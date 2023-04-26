# The xtrabackup Option Reference

This page documents all of the command-line options for the
`xtrabackup` binary.

## Options

### --apply-log-only
This option causes only the redo stage to be performed when preparing a
backup. It is very important for incremental backups.

### --backup
Make a backup and place it in `xtrabackup --target-dir`. See
[Creating a backup](../backup_scenarios/full_backup.md#creating-a-backup).

### --backup-lock-retry-count=#
The number of attempts to acquire metadata locks.

### --backup-lock-timeout=#
The timeout in seconds for attempts to acquire metadata locks.

### --binlog-info
This option controls how *Percona XtraBackup* should retrieve the server’s
binary log coordinates corresponding to the backup. Possible values are
`OFF`, `ON`, `LOCKLESS` and `AUTO`. See the *Percona XtraBackup*
[Lockless binary log information](../advanced/lockless_bin-log.md#lockless-bin-log)  manual page for more information.

### --check-privileges
This option checks if *Percona XtraBackup* has all the required privileges.
If a missing privilege is required for the current operation,
it will terminate and print out an error message.
If a missing privilege is not required for the current operation
but may be necessary for some other XtraBackup operation,
the process is not aborted, and a warning is printed.

```text
xtrabackup: Error: missing required privilege LOCK TABLES on *.*
xtrabackup: Warning: missing required privilege REPLICATION CLIENT on *.*
```

### --close-files
Do not keep files open. When *xtrabackup* opens a tablespace, it normally
does not close its file handle to manage the DDL operations
correctly. However, if the number of tablespaces is huge and can not
fit into any limit, there is an option to close file handles once they are
no longer accessed. *Percona XtraBackup* can produce inconsistent backups
with this option enabled. Use the option at your own risk.

### --compact
Create a compact backup by skipping secondary index pages.

### --compress
This option tells *xtrabackup* to compress all output data, including the
transaction log file and metadata files, using the specified compression
algorithm. The only currently supported algorithm is `quicklz`. The
resulting files have the qpress archive format.

Every `\*.qp` file
produced by xtrabackup is essentially a one-file qpress archive and can be
extracted and uncompressed by the [qpress](http://www.quicklz.com/)  file
archiver.

### --compress-chunk-size=#
Size of working buffer(s) for compression threads in bytes. The default
value is 64K.

### --compress-threads=#
This option specifies the number of worker threads used by *xtrabackup* for
parallel data compression. This option defaults to `1`. Parallel
compression (`xtrabackup --compress-threads`) can be used together
with parallel file copying (`xtrabackup --parallel`). For example,
`--parallel=4 --compress --compress-threads=2` will create 4 I/O threads
that will read the data and pipe it to 2 compression threads.

### --copy-back
Copy all the files in a previously made backup from the backup directory to
their original locations. This option will not copy over existing files
unless `xtrabackup --force-non-empty-directories` option is
specified.

### --core-file
Write core on fatal signals.

### --databases=#
This option specifies the list of databases and tables that should be backed
up. The option accepts the list of the form `"databasename1[.table_name1]
databasename2[.table_name2] . . ."`.

### --databases-exclude=name
Excluding databases based on name. This option operates the same way
as `xtrabackup --databases`, but matched names are excluded from the
backup. Note that this option has a higher priority than
`xtrabackup --databases`.

### --databases-file=#
This option specifies the path to the file containing the list of databases
and tables that should be backed up. The file can contain the list elements
of the form `databasename1[.table_name1]`, one element per line.

### --datadir=DIRECTORY
The source directory for the backup. This directory should be the same as the datadir
for your *MySQL* server, and it should be read from my.cnf if that
exists; otherwise, you must specify it on the command line.

When combined with the `xtrabackup --copy-back` or
`xtrabackup --move-back` option, `xtrabackup --datadir`
refers to the destination directory.

Once connected to the server, to perform a backup, you will need
`READ` and `EXECUTE` permissions at a filesystem level in the
server’s datadir.

### --debug-sleep-before-unlock=#
A debug-only option that is used by the Xtrabackup test suite.

### --decompress
This option decompresses all files with the `.qp` extension in a backup previously
made with the `xtrabackup --compress` option. The
`xtrabackup --parallel` option will allow multiple files to be
decrypted simultaneously. To decompress, the qpress utility MUST be
installed and accessible within the path. *Percona XtraBackup* doesn’t
automatically remove the compressed files. To clean up the backup
directory, users should use the `xtrabackup --remove-original` option.

### --decrypt=ENCRYPTION-ALGORITHM
Decrypts all files with the .xbcrypt extension in a backup
previously made with `xtrabackup --encrypt` option. The
`xtrabackup --parallel` option will allow multiple files to be
decrypted simultaneously. *Percona XtraBackup* doesn’t
automatically remove the encrypted files. To clean up the backup
directory, users should use `xtrabackup --remove-original` option.

### --defaults-extra-file=[MY.CNF]
Read this file after the global files are read. This file must be the first
option on the command-line.

### --defaults-file=[MY.CNF]
Only read default options from the given file. This file must be the first
option on the command-line and be a real file and cannot be a symbolic
link.

### --defaults-group=GROUP-NAME
This option sets up the group which should be read from the configuration
file. The option is used by the `--default-group` and is required for

`mysqld_multi` deployments.

### --defaults-group-suffix=#
Read the usual options groups and also groups with concat(group, suffix).


### --dump-innodb-buffer-pool
This option controls whether or not a new dump of buffer pool
content should be done.

With `--dump-innodb-buffer-pool`, *xtrabackup*
makes a request to the server to start the buffer pool dump (it
takes some time to complete and is done in background) at the
beginning of a backup provided the status variable
`innodb_buffer_pool_dump_status` reports that the dump has been
completed.

```shell
$ xtrabackup --backup --dump-innodb-buffer-pool --target-dir=/home/user/backup
```

By default, this option is set to OFF.

If `innodb_buffer_pool_dump_status` reports that there is running
dump of the buffer pool, *xtrabackup* waits for the dump to complete
using the value of –dump-innodb-buffer-pool-timeout

The file `ib_buffer_pool` stores tablespace ID and page ID
data used to warm up the buffer pool sooner.

### --dump-innodb-buffer-pool-timeout
This option contains the number of seconds that *xtrabackup* should
monitor the value of `innodb_buffer_pool_dump_status` to
determine if buffer pool dump has completed.

This option is used in combination with
`--dump-innodb-buffer-pool`. By default, it is set to 10
seconds.

### --dump-innodb-buffer-pool-pct
This option contains the percentage of the most recently used buffer pool
pages to dump.

This option is effective if –dump-innodb-buffer-pool option is set
to ON. If this option contains a value, *xtrabackup* sets the *MySQL*
system variable `innodb_buffer_pool_dump_pct`. As soon as the buffer pool
dump completes or it is stopped (see
–dump-innodb-buffer-pool-timeout), the value of the *MySQL* system
variable is restored.

### --encrypt=ENCRYPTION_ALGORITHM
This option instructs xtrabackup to encrypt backup copies of InnoDB data
files using the algorithm specified in the ENCRYPTION_ALGORITHM. It is
passed directly to the xtrabackup child process. See the
`xtrabackup`
[documentation](xtrabackup_binary.md) for more details.

### --encrypt-key=ENCRYPTION_KEY
This option instructs xtrabackup to use the given `ENCRYPTION_KEY` when
using the `xtrabackup --encrypt` option. It is passed directly to
the xtrabackup child process. See the xtrabackup
[documentation](xtrabackup_binary.md) for more details.

### --encrypt-key-file=ENCRYPTION_KEY_FILE
This option instructs xtrabackup to use the encryption key stored in the
given `ENCRYPTION_KEY_FILE` when using the `xtrabackup --encrypt`
option. It is passed directly to the xtrabackup child process. See the
xtrabackup [documentation](xtrabackup_binary.md) for more details.

### --encrypt-threads=#
This option specifies the number of worker threads that will be used for
parallel encryption/decryption.
See the xtrabackup [documentation](xtrabackup_binary.md) for more details.

### --encrypt-chunk-size=#
This option specifies the size of the internal working buffer for each
encryption thread, and is measured in bytes. It is passed directly to the
xtrabackup child process. See the `xtrabackup` [documentation](xtrabackup_binary.md) for more details.

!!! note

    To adjust the xbcloud/xbstream chunk size when you use encryption, you must adjust both the –encrypt-chunk-size and –read-buffer-size variables.

### --export
Create files necessary for exporting tables. See [Restoring Individual
Tables](restoring_individual_tables.md).

### --extra-lsndir=DIRECTORY
(for –backup): save an extra copy of the `xtrabackup_checkpoints`
and `xtrabackup_info` files in this directory.

### --force-non-empty-directories
When specified, it makes :option\`xtrabackup –copy-back\` and
`xtrabackup --move-back` option transfer files to non-empty
directories. No existing files will be overwritten. If files that need to
be copied/moved from the backup directory already exist in the destination
directory, it will still fail with an error.

### --ftwrl-wait-timeout=SECONDS
This option specifies time in seconds that xtrabackup should wait for
queries that would block `FLUSH TABLES WITH READ LOCK` before running it.
If there are still such queries when the timeout expires, xtrabackup
terminates with an error. The default is `0`, in which case it does not wait
for queries to complete and starts `FLUSH TABLES WITH READ LOCK`
immediately. Where supported (Percona Server 5.6+) xtrabackup will
automatically use [Backup Locks](https://www.percona.com/doc/percona-server/5.6/management/backup_locks.html#backup-locks)
as a lightweight alternative to `FLUSH TABLES WITH READ LOCK` to copy
non-InnoDB data to avoid blocking DML queries that modify InnoDB tables.

### --ftwrl-wait-threshold=SECONDS
This option specifies the query run time threshold which is used by
xtrabackup to detect long-running queries with a non-zero value of
xtrabackup –ftwrl-wait-timeout. `FLUSH TABLES WITH READ LOCK`
is not started until such long-running queries exist. This option has no
effect if xtrabackup –ftwrl-wait-timeout is `0`. The default value
is `60` seconds. Where supported (Percona Server 5.6+) xtrabackup will
automatically use [Backup Locks](https://www.percona.com/doc/percona-server/5.6/management/backup_locks.html#backup-locks)
as a lightweight alternative to `FLUSH TABLES WITH READ LOCK` to copy
non-InnoDB data to avoid blocking DML queries that modify InnoDB tables.

### --ftwrl-wait-query-type=all|update
This option specifies which types of queries are allowed to complete before
xtrabackup will issue the global lock. The default is `all`.

### --galera-info
This options creates the `xtrabackup_galera_info` file which contains
the local node state at the time of the backup. Option should be used when
performing the backup of Percona XtraDB Cluster. It has no effect when
backup locks are used to create the backup.

### --generate-new-master-key
Generates a new master key when doing a copy-back operation.

### --history=name
This option enables the tracking of the backup history in the
`PERCONA_SCHEMA.xtrabackup_history` table. An optional history series name
may be specified that will be placed with the history record for the backup
being taken.

### --incremental
This option tells xtrabackup to create an incremental backup. It is passed to
the xtrabackup child process. When this option is specified, either
`xtrabackup --incremental-lsn` or xtrabackup
–incremental-basedir can also be given. If neither option is given, option
`xtrabackup --incremental-basedir` is passed to xtrabackup by
default, set to the first timestamped backup directory in the backup base
directory.

### --incremental-basedir=DIRECTORY
This directory contains the full backup, which is the base dataset used for the incremental backups.

### --incremental-dir=DIRECTORY
When preparing an incremental backup, this is the directory where the
incremental backup is combined with the full backup to make a new full
backup.

### --incremental-force-scan
When creating an incremental backup, force a full scan of the data pages in
the instance to be used in the backup even if the complete changed page bitmap data is
available.

### --incremental-history-name=name
This option specifies the name of the backup series stored in the
[PERCONA_SCHEMA.xtrabackup_history](../innobackupex/storing_history.md#xtrabackup-history) history record
to base an incremental backup on. *xtrabackup* searches the history
table for the most recent (highest innodb_to_lsn), successful backup
in the series and take the to_lsn value to use as the starting lsn for the
incremental backup. This will be mutually exclusive with `xtrabackup
--incremental-history-uuid`, `xtrabackup --incremental-basedir` and
`xtrabackup --incremental-lsn`. If no valid `LSN` can be found
(no series by that name, no successful backups by that name) *xtrabackup*
returns an error. It is used with the `xtrabackup --incremental`
option.

### --innodb-checksum-algorithm=name
The algorithm InnoDB uses to calculate a page checksum. The available
algorithms are CRC32, INNODB, NONE, STRICT_CRC32, STRICT_INNODB,
and STRICT_NONE

### --incremental-history-uuid=UUID
This option specifies the `UUID` of the specific history record stored
in the [PERCONA_SCHEMA.xtrabackup_history](../innobackupex/storing_history.md#xtrabackup-history) to base
an incremental backup on `xtrabackup --incremental-history-name`,
`xtrabackup --incremental-basedir` and `xtrabackup --incremental-lsn`. If no valid LSN is found (no success record with
that UUID) *xtrabackup* returns an error. This option is used with
the `xtrabackup --incremental` option.

### --incremental-lsn=LSN
When creating an incremental backup, you can specify the log sequence number
(`LSN`) instead of specifying
`xtrabackup --incremental-basedir`. For databases created in 5.1 and
later, specify the LSN as a single 64-bit integer. **ATTENTION**: If
a wrong LSN value is specified (a user  error that *Percona XtraBackup* cannot detect), the backup will be unusable. Be careful!

### --innodb-log-arch-dir=DIRECTORY
This option is used to specify the directory containing the archived logs.
It can only be used with the `xtrabackup --prepare` option.

### --innodb-miscellaneous
A large group of InnoDB options are normally read from the
`my.cnf` configuration file, so that *xtrabackup* boots up its
embedded InnoDB in the same configuration as your current server. You
normally do not need to specify these explicitly. These options have the
same behavior that they have in InnoDB or XtraDB. They are as follows:

```text
--innodb-adaptive-hash-index
--innodb-additional-mem-pool-size
--innodb-autoextend-increment
--innodb-buffer-pool-size
--innodb-checksums
--innodb-data-file-path
--innodb-data-home-dir
--innodb-doublewrite-file
--innodb-doublewrite
--innodb-extra-undoslots
--innodb-fast-checksum
--innodb-file-io-threads
--innodb-file-per-table
--innodb-flush-log-at-trx-commit
--innodb-flush-method
--innodb-force-recovery
--innodb-io-capacity
--innodb-lock-wait-timeout
--innodb-log-buffer-size
--innodb-log-files-in-group
--innodb-log-file-size
--innodb-log-group-home-dir
--innodb-max-dirty-pages-pct
--innodb-open-files
--innodb-page-size
--innodb-read-io-threads
--innodb-write-io-threads
```

### --innodb-undo-directory=name
The directory location for the undo tablespace. The path is absolute.

### --innodb-undo-tablespace=#
The number of undo tablespaces to use.

### --keyring-file-data=FILENAME
The path to the keyring file. Combine this option with
`xtrabackup --xtrabackup-plugin-dir`.

### --kill-long-queries-timeout=#
This options specifies the number of seconds xtrabackup waits between
starting FLUSH TABLES WITH READ LOCK and killing those queries that block
it. The default is `0` (zero) seconds, which means the xtrabackup does not
attempt to kill any queries.

### --kill-long-query-type=select|all
This option specifies which types of queries should be killed to unblock the global lock. The default value is `select`.

### --lock-ddl
Issue `LOCK TABLES FOR BACKUP` if it is supported by server
at the beginning of the backup to block all DDL operations.

### --lock-ddl-per-table
Lock DDL for each table before xtrabackup starts to copy
it and until the backup is completed.

### --lock-ddl-timeout
If `LOCK TABLES FOR BACKUP` does not return within given
timeout, abort the backup.

### --log-bin[=name]
The base name for the log sequence.

### --log-copy-interval=#
This option specifies time interval between log copying
thread checks in milliseconds (default is 1 second).

### --login-path=#
Read this path from the login file.

### --move-back
Move all the files in a previously made backup from the backup directory to
their original locations. As this option removes backup files, it must be
used with caution.

### --no-backup-locks
This options controls if backup locks are used instead of `FLUSH TABLES
WITH READ LOCK` during the backup stage. The backup locks are must be supported on the server for the option to have an affect.

This option is enabled by default. Disable the option
with `--no-backup-locks`.

### --no-defaults
Do not read the default options from any option file. Must be given as the first
option on the command-line.

### --no-lock
This option automatically uses Backup Locks, and disables table locks, as a
lightweight alternative to `FLUSH TABLES WITH READ LOCK` to copy
non-InnoDB data to avoid blocking DML queries that modify InnoDB tables.

Only use this option if *all* tables are InnoDB and you *do not care* about
the binary log position of the backup.

Do not use this option if any DDL statements will be executed or if any
non-InnoDB tables are being updated (this includes the MyISAM tables in the
mysql database). Using this option in these conditions could cause an
inconsistent backup.

If your backups fail to acquire a lock and you are planning to use this
option, the failure may be caused by incoming replication events that
prevent the lock from succeeding. Try the `--safe-slave-backup`
to momentarily stop the replication slave thread.

The `xtrabackup-binlog-info` is not created when the `--no-lock`
is used because `SHOW MASTER STATUS` may be inconsistent. In certain
conditions, `xtrabackup_binlog_pos_innodb` can be used instead to get
consistent binlog coordinates as described in [Working with Binary Logs](working_with_binary_logs.md#working-with-binlogs).

### --no-version-check
This option disables the version check. If you do not pass this option, the
automatic version check is enabled implicitly when xtrabackup runs
in the `--backup` mode. To disable the version check, explicitly pass
the `--no-version-check` option when invoking xtrabackup.

When the automatic version check is enabled,xtrabackup performs a
version check against the server on the backup stage after creating a server
connection. xtrabackup sends the following information to the server:

* MySQL flavour and version

* Operating system name

* Percona Toolkit version

* Perl version

Each piece of information has a unique identifier which is an MD5 hash value
that Percona Toolkit uses to obtain statistics about how it is used. This value is
a random UUID; no client information is either collected or stored.

### --open-files-limit=#
The maximum number of file descriptors to reserve with setrlimit().

### --parallel=#
This option specifies the number of threads to use to copy multiple data
files concurrently when creating a backup. The default value is 1 (i.e., no
concurrent transfer). In *Percona XtraBackup* 2.3.10 and newer, this option
can be used with `xtrabackup --copy-back` option to copy the user
data files in parallel (redo logs and system tablespaces are copied in the
main thread).

### --password=PASSWORD
This option specifies the password to use when connecting to the database.
It accepts a string argument. See `mysql --help` for details.

### --prepare
Makes `xtrabackup` perform recovery on a backup created with
`xtrabackup --backup`, so that it is ready to use. See
[preparing a backup](../backup_scenarios/full_backup.md#preparing-a-backup).

### --print-defaults
Print the program argument list and exit. Must be given as the first option
on the command-line.

### --print-param
Makes `xtrabackup` print out parameters that to copy the data files back to their original locations to restore them. See
[Scripting Backups With xtrabackup](scripting_backups_xbk.md#scripting-xtrabackup).

### --read-buffer-size[=#]
Set read buffer size. The given value is scaled up to page size. The
default is 10MB.

Use this variable to increase the xbcloud/xbstream chunk size from the default value of 10MB.

**NOTE**: When you use encryption, to adjust the xbcloud/xbstream chunk size, adjust both the `--encrypt-chunk-size` and `--read-buffer-size` variables.

```sql
$ xtrabackup ... --read-buffer-size=1G | xbcloud put ...
```

### --rebuild-indexes
Rebuild secondary indexes in InnoDB tables after applying the log. Only use
with –prepare.

### --rebuild-threads=#
This option defines the number of threads to rebuild indexes in a compact
backup. Only use with `--prepare` and `--rebuild-indexes`.

### --redo-log-version=#
The redo log version of the backup. Use only with `--prepare`.

### --reencrypt-for-server-id=<new_server_id>
Use this option to start the server instance with different server_id from
the one the encrypted backup was taken from, like a replication replica or a
Galera node. When this option is used, xtrabackup will, as a prepare step,
generate a new master key with ID based on the new server_id, store it into
keyring file, and re-encrypt the tablespace keys inside of tablespace
headers. The option should be passed for `--prepare` (final step).

### --remove-original
Implemented in *Percona XtraBackup* 2.4.6, this option when specified will
remove `.qp`, `.xbcrypt` and `.qp.xbcrypt` files after
decryption and decompression.

### --rsync
Use the `rsync` utility to optimize local file transfers.

When this option is specified, xtrabackup uses `rsync` to copy all
non-InnoDB files instead of spawning a separate copy command for each file.
This option is faster for servers with a large number of databases or tables.

This option cannot be used with `--stream`.

### --safe-slave-backup
When specified, xtrabackup will stop the replica SQL thread just before
running `FLUSH TABLES WITH READ LOCK` and wait to start backup until
`Slave_open_temp_tables` in `SHOW STATUS` is zero. If there are no open
temporary tables, the backup will occur; otherwise the SQL thread will
be started and stopped until there are no open temporary tables. The backup
will fail if `Slave_open_temp_tables` does not become zero after
xtrabackup –safe-slave-backup-timeout seconds. The replica SQL
thread will be restarted when the backup finishes. This option is
implemented to deal with [replicating temporary tables](https://dev.mysql.com/doc/refman/5.7/en/replication-features-temptables.html)
and isn’t neccessary with Row-Based-Replication.

### --safe-slave-backup-timeout=SECONDS
How many seconds `xtrabackup --safe-slave-backup` should wait for
`Slave_open_temp_tables` to become zero. The default is 300 seconds.

### --secure-auth
Refuse client connecting to the server if it uses old (pre-4.1.1) protocol.
(Enabled by default; use –skip-secure-auth to disable.)

### --server-id=#
The server instance being backed up.

### --server-public-key-path=name
File path the server’s public RSA key in PEM format.

### --skip-tables-compatibility-check
This option disables the engine compatibility warning.

!!! admonition "See also"

    `--tables-compatibility-check`

### --slave-info
This option is useful when backing up a replication replica server. It prints
the binary log position of the source server. It also writes the binary log
coordinates to the `xtrabackup_slave_info` file as a `CHANGE MASTER`
command. A new replica for this source can be set up by starting a replica server
on this backup and issuing a `CHANGE MASTER` command with the binary log
position saved in the xtrabackup_slave_info file.

### --ssl
Enable secure connection. More information can be found in [–ssl](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-secure-connections.html)
MySQL server documentation.

### --ssl-ca
Path of the file, which contains a list of trusted SSL CAs. More information
can be found in [–ssl-ca](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-secure-connections.html)
MySQL server documentation.

### --ssl-capath
The directory path that contains trusted SSL CA certificates in the PEM format. More
information can be found in [–ssl-capath](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-secure-connections.html)
MySQL server documentation.

### --ssl-cert
Path of the file which contains X509 certificate in PEM format. More
information can be found in [–ssl-cert](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-secure-connections.html)
MySQL server documentation.

### --ssl-cipher
List of permitted ciphers to use for connection encryption. More information
can be found in [–ssl-cipher](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-secure-connections.html)
MySQL server documentation.

### --ssl-crl
Path of the file that contains certificate revocation lists. More
information can be found in [–ssl-crl](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-secure-connections.html)
MySQL server documentation.

### --ssl-crlpath
Path of the directory that contains certificate revocation list files. More
information can be found in [–ssl-crlpath](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-secure-connections.html)
MySQL server documentation.

### --ssl-key
Path of the file that contains X509 key in PEM format. More information can be
found in [–ssl-key](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-secure-connections.html)
MySQL server documentation.

### --ssl-mode
The security state of connection to server. More information can be found in
[–ssl-mode](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-secure-connections.html)
MySQL server documentation.

### --ssl-verify-server-cert
Verify server certificate Common Name value against host name used when
connecting to server. More information can be found in
[–ssl-verify-server-cert](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-secure-connections.html)
MySQL server documentation.

### --stats
Causes xtrabackup to scan the specified data files and print out
index statistics.

### --stream=name
Stream all backup files to the standard output in the specified format.
Currently supported formats are `xbstream` and `tar`.

### --tables=name
A regular expression against which the full tablename, in
`databasename.tablename` format, is matched. If the name matches, the
table is backed up. See [partial backups](partial_backups.md).

### --tables-compatibility-check
This option enables the engine compatibility warning.

The default value is `ON`. Use `--skip-tables-compatibility-check`
to disable.

### --tables-exclude=name
Filtering by regexp for table names. Operates the same
way as `xtrabackup --tables`, but matched names are excluded from
backup. Note that this option has a higher priority than
`xtrabackup --tables`.

### --tables-file=name
A file containing one table name per line, in databasename.tablename format.
The backup will be limited to the specified tables. See
[Scripting Backups With xtrabackup](scripting_backups_xbk.md#scripting-xtrabackup).

### --target-dir=DIRECTORY
This option specifies the destination directory for the backup. If the
directory does not exist, `xtrabackup` creates it. If the directory
does exist and is empty, `xtrabackup` will succeed.
`xtrabackup` will not overwrite existing files; however it will
fail with operating system error 17, `file exists`.

If this option is a relative path, it is interpreted as being relative to
the current working directory from which xtrabackup is executed.

In order to perform a backup, you need `READ`, `WRITE`, and `EXECUTE`
permissions at a filesystem level for the directory that you supply as the
value of `--target-dir`.

### --throttle=#
This option limits the number of chunks copied per second. The chunk size is
*10 MB*.

To limit the bandwidth to *10 MB/s*, set the option to *1*:
`--throttle=1`.

### --tls-version=name
The TLS version to use. The allowed values are the following:

* TLSv1

* TLSv1.1

* TLSv1.2

### --tmpdir=name
This option is currently not used for anything except printing out the
correct tmpdir parameter when `xtrabackup --print-param` is used.

### --to-archived-lsn=LSN
This option is used to specify the LSN to which the logs should be applied
when backups are being prepared. It can only be used with the
`xtrabackup --prepare` option.

### --transition-key
This option is used to enable processing the backup without accessing the
keyring vault server. In this case, xtrabackup derives the AES
encryption key from the specified passphrase and uses it to encrypt
tablespace keys of tablespaces being backed up.

If `--transition-key <xtrabackup –transition-key>` does not have any
value, xtrabackup will ask for it. The same passphrase should be
specified for the `xtrabackup --prepare` command.

### --use-memory=#
This option affects how much memory is allocated and is similar to `innodb_buffer_pool_size`. This option is only relevant in the `--prepare` phase or when analyzing statistics with `--stats`. The default value is 100MB. The recommended value is between 1GB to 2GB. Multiples are supported providing the unit (for example, 1MB, 1M, 1GB, 1G).

### --user=USERNAME
This option specifies the MySQL username used when connecting to the server,
if that’s not the current user. The option accepts a string argument. See
mysql –help for details.

### --version
This option prints *xtrabackup* version and exits.

### --xtrabackup-plugin-dir=DIRNAME
The absolute path to the directory that contains the `keyring` plugin.

!!! admonition "See also"

    Percona Server for MySQL Documentation: keyring_vault plugin with Data at Rest Encryption
    https://www.percona.com/doc/percona-server/5.7/security/data-at-rest-encryption.html
    MySQL Documentation: Using the keyring_file File-Based Plugin
    https://dev.mysql.com/doc/refman/5.7/en/keyring-file-plugin.html