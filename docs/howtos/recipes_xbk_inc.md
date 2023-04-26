# Make an incremental backup

Backup all the InnoDB data and log files - located in `/var/lib/mysql/` -
**once**, then make two daily incremental backups in `/data/backups/mysql/`
(destination). Finally, prepare the backup files to be ready to restore or use.

## Create one full backup

Making an incremental backup requires a full backup as a base:

```shell
$ xtrabackup --backup --target-dir=/data/backups/mysql/
```

It is important that you **do not run** the `xtrabackup --prepare` command yet.

## Create two incremental backups

Suppose the full backup is on Monday, and you will create an incremental one on Tuesday:

```shell
$ xtrabackup --backup --target-dir=/data/backups/inc/tue/ \
      --incremental-basedir=/data/backups/mysql/
```

and the same policy is applied on Wednesday:

```shell
$ xtrabackup --backup --target-dir=/data/backups/inc/wed/ \
       --incremental-basedir=/data/backups/inc/tue/
```

## Prepare the base backup

Prepare the base backup (Monday’s backup):

```shell
$ xtrabackup --prepare --apply-log-only --target-dir=/data/backups/mysql/
```

## Roll forward the base data to the first increment

Roll Monday’s data forward to the state on Tuesday:

```shell
$ xtrabackup --prepare --apply-log-only --target-dir=/data/backups/mysql/ \
   --incremental-dir=/data/backups/inc/tue/
```

## Roll forward again to the second increment

Roll forward again to the state on Wednesday (without --apply-log-only):

```shell
$ xtrabackup --prepare --target-dir=/data/backups/mysql/ \
   --incremental-dir=/data/backups/inc/wed/
```

## Notes

* You might want to set the `xtrabackup --use-memory` to speed up the
process if you are on a dedicated server that has enough free memory. More
details [here](../xtrabackup_bin/xbk_option_reference.md).

* A more detailed explanation is [here](../xtrabackup_bin/incremental_backups.md).