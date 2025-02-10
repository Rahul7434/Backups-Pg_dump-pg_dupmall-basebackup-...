# Backup
In PostgreSQL, there are two types of Backups logical and Physical.
1. logical: -
    • Pg_dump & pg_dumpall
2. Physical Backup: -
    • Pg_basebackup
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
1. Pg_dump:
  - Creates the logical backup which means it provides multiple formats to take backups that contain SQL commands (DDL, DML). Or data in a portable format like “custom, tar”. This allows backup to be restored on different Matchine or Postgresql versions.
  -while the backup is running “pg_dump” takes a snapshot of the database, so changes made during the dump process won’t affect the backup.
  - pg_dump works on one database at a time.
  - We can back up the single “Database, table, schema”.
```
  Pg_dump Formats: - -F, --format=c|d|t|p: Specifies the output format
    • Plain\sql:- p (plain): SQL script output (default).
  [pg_dump –h hostname –U username –p port –d database -f backup.sql]
    • Tar:- t (tar): Outputs in tar archive format.
  [pg_dump –h hostname –U username –p port –d database -F t -f backup.tar]
    • Custom:- c (custom): The most flexible format, suitable for pg_restore.
  [pg_dump –h hostname –U username –p port –d database -F c -f backup.dump]
    • Directory:- d (directory): Dumps the database into a directory with one file per table.
  [pg_dump –h hostname –U username –p port –d database -F d -f backup.dir]
```
```
Dump Options:-
    • -a, --data-only: Dumps only the data, not the schema.
        ◦ [pg_dump -U username –h host –p port –a -d database_name  -F p/t/d/c –f  data_only_backup.bak]
    • -s, --schema-only: Dumps only the schema, not the data.
        ◦ [pg_dump -U username –h host –p port -d database_name -s  -F p/t/d/c –f  data_only_backup.sql]

    • -t, --table=TABLE: Dumps only the specified table(s). It can be used multiple times for multiple tables.
        ◦ [pg_dump -U username –h host –p port -d database_name –t tablename  -F p/t/d/c –f  data_only_backup.baks]

    • -n, --schema=SCHEMA: Dumps only the specified schema(s).
        ◦ [pg_dump -U username –h host –p port -d database_name -n schemaname -F p/t/d/c –f  data_only_backup.bak]

    • -T, --exclude-table=TABLE: Excludes the specified table(s) from the dump.
        ◦ [pg_dump -U username –h host –p port  -d database_name –T tablename  -F p/t/d/c –f  data_only_backup.bak]

    • -N, --exclude-schema=SCHEMA: Excludes the specified schema(s) from the dump.
        ◦ [pg_dump -U username –h host –p port  -d database_name –N schemaname  -F p/t/d/c –f  data_only_backup.bak]

    • -x, --no-privileges: Does not dump GRANT/REVOKE statements.
        ◦ [pg_dump -U username –h host –p port  -d database_name -x -F p/t/d/c –f  data_only_backup.bak]

    • --no-owner: Do not dump ownership information.
        ◦ [pg_dump -U username –h host –p port –a -d database_name --no -F p/t/d/c –f  data_only_backup.bak]

    • -O, --no-reconnect: Prevents reconnection to the database.
        ◦ [pg_dump -U username –h host –p port  -d database_name -O -F p/t/d/c –f  data_only_backup.bak]
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
2. Pg_dumpall: - 
    • pg_dumpall is ideal for taking full cluster backups, including all databases and global objects like users and roles.
    • It creates a plain-text SQL script that can be restored with psql.
    • You can back up global objects only (-g) or all databases together.
```
  1.Here is an example of how to use pg_dumpall to take a full backup:
  [pg_dumpall -h hostname -p port -U username -f cluster_backup.sql]
```
```
2.pg_dumpall -g -f globals_only.sql
The -g option tells pg_dumpall to dump only global objects (roles, users, tablespaces) and skip individual 
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
3. Pg_basebackup: - 
  -Physical Backup: Creates a binary copy of the entire PostgreSQL data directory, including all data files, configuration files, and transaction logs.
  -Often used in replication setups to create a base backup for streaming replication.
  Ensures a consistent backup by including the necessary WAL (Write-Ahead Logging) files.
  -You can choose various output formats, compression, and exclude certain files (e.g., unlogged tables).
- It supports taking backups from a running (online) PostgreSQL server without taking it offline.
```
[pg_basebackup -h <host> -U <replication_user> -D /path/to/backup_directory -Fp -Xs –P]
```
```
  Key Options:
    • -h: Specifies the host where the PostgreSQL instance is running.
    • -U: Specifies the replication user with appropriate privileges (usually with the REPLICATION role).
    • -D: Specifies the directory where the backup will be stored.
    • -Fp: Choose plain file format (as opposed to tar format).
    • -Xs: Includes WAL files necessary for recovery.
    • -P: Provides progress information during the backup process.
```
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Restoration
We can restore the backups using “psql”, “\i”, and “pg_restore’ utilities.
-if the backup is in plain format then it should RESTORE only with psql utility.
-if the backup is in custom, tar, or Directory format then it should RESTORE only with the pg_restore utility.

###### Pg_dump Restoration:-
```
* pg_dump IN PLAIN –F p Backup:
	[psql -h hostname -U username -d database_name -f backup.sql]
	
	Using \i command inside the psql
	[\i 'path/to/backup_file.sql']
```
###### Pg_dumpall Restoration:-
```
* pg_dumpall Backup:
To restore the full cluster backup you can use psql:
	[psql -h hostname -U username -f cluster_backup.sql]
```
```
* pg_dumpall IN CUSTOM –F c Backup:
	[pg_restore -h hostname -U username -d database_name -F c backup.dump]

*If you want to drop and recreate the database before restoring:
  pg_restore -h hostname -U username -d database_name --clean --create -F c backup.dump 
```
```
* pg_dumpall IN CUSTOM –F t Backup:
	[pg_restore -h hostname -U username -d database_name -F t backup.tar]
```
```
* pg_dumpall IN CUSTOM –F d Backup:
		pg_restore -h hostname -U username -d database_name -F d /path/to/directory/backup.tar
```
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
##### Restoring from a Physical Backup (base_backup)
To restore a physical backup, the steps involve:
```
    1. Stopping the PostgreSQL server.
    2. Copying the contents of the backup directory to the PostgreSQL data directory (or extracting the tar files if backup was taken in tar format).
    3. Ensuring the WAL files are available for recovery.
    4. Starting the PostgreSQL server.
```
