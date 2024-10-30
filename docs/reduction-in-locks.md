# Reduced backup lock time

--8<--- "pro-build-announcement.md"

!!! important

    The `--lock-ddl=REDUCED` option is a [tech preview](./glossary.md#tech-preview). Before using this option in production, we recommend that you test restoring production from physical backups in your environment, and also use the alternative backup method for redundancy.

Percona Server for MySQL implemented `LOCK TABLES FOR BACKUP` to block Data Definition Language (DDL) statements on a server. Percona XtraBackup utilizes this lock during the backup process to ensure that any `DDL` event does not corrupt the backup. The lock does not affect the Data Manipulation Language (DML) statements.

## How the backup lock works

With the [`--lock-ddl=ON`](./xtrabackup-option-reference.md/#lock-ddl) option the backup lock is enabled by default from the start of the backup process.

* Set a backup lock on the server to prevent any `DDL` operations to ensure no new tables are created, renamed, or dropped during the backup.

* Copy redo logs from the checkpoint up to the current Log Sequence Number (LSN) and start tracking new entries.

* Identify and copy the `.ibd` files.

* Copy non-InnoDB files.

* Gather a sync point from all engines (InnoDB LSN, binlog, Global Transaction Identifier (GTID), etc.) by querying the log status.

* Stop the redo thread once it copies at least up to sync point.

* Release the backup lock on the server.

If the backup lock is disabled with the [`--lock-ddl=OFF`](./xtrabackup-option-reference.md/#lock-ddl) option, a backup continues while concurrent DDL events are executed. These backups may be invalid and may fail at either the `backup` or the `--prepare` step.

## The `--lock-ddl=REDUCED` option overview

[Percona XtraBackup 8.4.0-2](./release-notes/8.4.0-2.md) adds the [`--lock-ddl=REDUCED`](./xtrabackup-option-reference.md/#lock-ddl) option to reduce the time the server remains locked by `xtrabackup`. Now, you can execute more `DDL` operations concurrently while the backup is in progress. The backup lock is now taken after copying the `.ibd` files and before copying the `non-InnoDB` files.

### Benefits

The `--lock-ddl=REDUCED` option key features are as follows:

* Reduced locking: Rather than holding the backup lock throughout the entire backup process, it is only acquired for a very short period.

* DDL statements: The server remains accessible for `DDL` operations during the backup process.

* Consistency: Consistency is maintained by using the redo log to track file operations. During the backup process, new metadata files are created to ensure consistency. In the `--prepare` phase, these metadata files are processed first to bring the database to a consistent state.

The new backup design includes the following phases:

### Phase 1: `Operations performed without the lock`

1. Copy the redo logs from the checkpoint up to the current `LSN` and start tracking new entries.
2. Start the redo log thread.
3. Track file operations from the redo log by parsing the `MLOG_FILE_*` records.
4. Copy the `.ibd` files.

### Phase 2: `Operations performed under the lock`

1. Acquire the backup lock (Lock Tables For Backup (LTFB)/Lock Instance For Backup (LIFB)) to prevent new DDL operations, such as creating or altering tables.
2. Check the file operations that were tracked and recopy the tablespaces.
3. Create additional `meta` files to perform the required actions (deletions or renames) on the already copied files. This approach ensures that the backup remains consistent and accurate without disrupting the streaming process. This step is required for taking streaming backups.
    
    The meta files are the following:

    | File         | Description                                                                                                      |
    |--------------|------------------------------------------------------------------------------------------------------------------|
    | `.new`       | For files that need to be recopied due to encryption or `ADD INDEX` operations.                                  |
    | `.del`       | For deleted files. If a file has been copied, create a `space_id.del` file.                                      |
    | `.ren`       | For renamed files after copying. Create a `space_id.ren` file.                                                   |
    | `.crpt`      | For files that cannot be fully copied due to encryption changes. These will be recopied, and a `.crpt` file will be created for the existing file. |
    | `.new.meta`  | Similar to `.new`, but for incremental backups.                                                                  |
    | `.new.delta` | Similar to `.new`, but for incremental backups. Create `t1.ibd.meta` and `t1.ibd.delta` instead of `t1.ibd`.      |

4. Gather a sync point from all engines (InnoDB LSN, binlog, GTID, etc.) by querying the `log_status`.
5. Stop the redo thread once it copies at least up to sync point at step 4.
6. Release the backup lock (LTFB/LIFB).

### Phase 3. Processing the new metadata files during the `--prepare` phase before crash recovery starts.

1. The `.crpt` files are removed matching the name after stripping the extension. This step is crucial before the IBD scan because these files are incomplete (they could even be zero size).
2. Perform a scan to create a mapping of `space_id` to file names.
3. The `space_id.del` deletes the file matching the `space_id`. In case of incremental backups, also deletes the corresponding `.new.meta` and `.new.delta` files.
4. The `space_id.ren` renames the file matching the `space_id` to the name specified in the file.
5. The `.new` extension replaces the file that matches the name without the `.new` extension.
6. The `.new.meta` and `.new.delta` replace the meta and delta files that match the name without the `.new` in the name.

After Phase 3 is completed, the regular crash recovery starts.

### Limitations

1. The `ALTER INSTANCE ROTATE MASTER KEY` command should not be executed during [Phase 1](#phase-1-operations-performed-without-the-lock) if there are encrypted tables, as it will cause the backup to fail.

2. The number of open file handles in your Operation System (OS) should be configured to match the number of files in the server data directory.

3. When using the `--lock-ddl=REDUCED` option, backups may be larger because new `DDL` operations are executed concurrently on the server, and the files they generate are included in the backup. Additionally, certain DDL operations, such as `ADD INDEX` or encryption changes to existing data files, will cause these data files to be recopied, further increasing the backup size.










