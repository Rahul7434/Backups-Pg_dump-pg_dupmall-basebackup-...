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
        ◦ [pg_dump -U username –h host –p port -d database_name --no-owner -F p/t/d/c –f  data_only_backup.bak]

    --section=section: Dump only a specific section (pre-data, data, post-data).		
		pre-data	Definitions like tables, types, functions, schemas, and basic structure
		data	Actual table contents, large objects, and sequence values
		post-data	Indexes, triggers, rules, constraints (like primary keys, foreign keys, etc.)

    	[pg_dump -U username -d dbname --section=pre-data --section=data -F p -f partial_dump.sql]

🧹 Metadata Control
--no-owner: Skip ownership commands (ALTER OWNER).

--no-acl: Skip access privileges (GRANT, REVOKE).

--inserts: Use INSERT statements instead of COPY.

--column-inserts: Use INSERT with column names.

--no-comments: Exclude comments from the dump.

--no-security-labels: Skip security labels.

--no-subscriptions: Skip logical replication subscriptions.

🔄 Restore Behavior
-c or --clean: Add DROP statements before CREATE.

-C or --create: Include CREATE DATABASE statement.

--if-exists: Use DROP IF EXISTS for objects.

🚀 Performance & Parallelism
-j njobs or --jobs=njobs: Run parallel jobs (only with directory format).

--lock-wait-timeout=ms: Set timeout for locking objects.

🌐 Encoding & Extensions
-E encoding or --encoding=encoding: Set output encoding.

-e extension or --extension=extension: Dump only specific extension(s).

🧪 Advanced Filtering
--exclude-table-data=table: Exclude data for specific table(s).

--include-foreign-data=server: Include foreign tables from a server.

--enable-row-security: Include rows protected by row-level security.
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

```
* pg_dumpall Backup:
To restore the full cluster backup you can use psql:
	[psql -h hostname -U username -f cluster_backup.sql]
```
```
* pg_dump IN CUSTOM –F c Backup:
	[pg_restore -h hostname -U username -d database_name -F c backup.dump]

*If you want to drop and recreate the database before restoring:
  pg_restore -h hostname -U username -d database_name --clean --create -F c backup.dump 
```
```
* pg_dump IN CUSTOM –F t Backup:
	[pg_restore -h hostname -U username -d database_name -F t backup.tar]
```
```
* pg_dump IN CUSTOM –F d Backup:
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
---
---
# Maintenance-Task

##### Vacuum
- PostgreSQL VACUUM Overview:
      - PostgreSQL uses the MVCC (Multi-Version Concurrency Control) system for handling transactions, which allows multiple versions of rows in a table. When rows are updated or deleted, old row versions (or "dead tuples") stay in the table until they are cleaned up. The VACUUM process helps in cleaning these dead tuples to free up space and maintain database performance.

- There are three types of vacuum types: 
    1. Regular VACUUM:
	      Cleans up dead tuples but does not reclaim disk space. Instead, it marks the space as reusable.
    2. VACUUM FULL:
	     This fully rewrites the table to remove dead space and return unused disk space to the operating system. However, it locks the table during the operation.
- Important Parameters of VACUUM:
  ```
    1. autovacuum:
	      A background process that automatically runs VACUUM to maintain the database without manual intervention. Configurable via parameters in the postgresql.conf file.
  ```

- The visibility of pages determines whether or not each page has dead tuples. Vacuum processing can skip a page that does not have dead tuples by using the corresponding visibility map (VM).
###### Workflow of VACUUM in PostgreSQL:
1. Marking Dead Tuples:
```
	When a row is updated or deleted, the old version of the row is marked as "dead."
	VACUUM identifies these dead tuples and marks them for removal.
	VACUUM scans the entire table to identify and clean up dead tuples.
	It reads data pages to determine if any dead tuples are present.
```
3. Freeing Space:
```
	Once dead tuples are found, they are removed, and the space is marked as free.
	This space can now be reused for new data inserts or updates.
```
5. Updating Visibility Map:
```
VACUUM updates the visibility map to mark which pages contain only visible (live) tuples, helping future scans avoid unnecessary reads.
```
7. Transaction IDs Wraparound Prevention:
```
Regular VACUUMing also prevents transaction ID wraparound by updating the pg_class and pg_stat_activity to maintain healthy transaction counters.
```
2. vacuum_cost_delay:
```
	Specifies the delay between VACUUM operations to reduce I/O load on the system.
```
4. vacuum_freeze_min_age:
```
	Controls the minimum transaction age at which tuples are frozen to avoid transaction ID wraparound.
```
6. vacuum_defer_cleanup_age:
```
	This sets the number of transactions that can defer tuple cleanup to help reduce conflicts with long-running transactions.
```
8. vacuum_cost_limit:
```
	Determines the limit of I/O operations a VACUUM process can perform before pausing to avoid system overload.
```
-----------------------------------------------------------------------------------------------------------------------------------------
##### Internal Working of VACUUM:
1. Visibility Map and Free Space Map:
	VACUUM updates the Visibility Map, allowing PostgreSQL to track pages with only live tuples for future optimizations.
	The Free Space Map (FSM) tracks which pages have space available for inserts, reducing the need for scanning the entire table.
2. Multi-Version Concurrency Control (MVCC):
	When rows are updated or deleted, MVCC keeps the older versions of the data for transaction consistency. VACUUM removes these old versions that are no longer needed.

4. Importance of autovacuum: 
	Highlight the role of the automatic autovacuum process in preventing performance degradation and wraparound issues without manual intervention.
5. Parameters:
	Mention key parameters that can be tuned for better performance, like vacuum_cost_delay (to reduce I/O impact) and vacuum_freeze_min_age (for freezing old tuples).
----------------------------------------------------------------------------------------------------------------------------
- Syntax:- 
1. Basic VACUUM
```
VACUUM; , VACUUM <table_name>;
```
2. VACUUM FULL
```
VACUUM FULL;  , VACUUM FULL <table_name>;
```
3. VACUUM with ANALYZE
```
VACUUM ANALYZE; , VACUUM ANALYZE <table_name>;
```
4. VACUUM with VERBOSE
```
VACUUM VERBOSE; , VACUUM VERBOSE <table_name>;
```
5. VACUUM FREEZE
```
VACUUM FREEZE; , VACUUM FREEZE <table_name>;
```
6. VACUUM FULL with ANALYZE
```
VACUUM FULL ANALYZE;  VACUUM FULL ANALYZE <table_name>;
```
7. VACUUM with Specific Columns for ANALYZE
```
VACUUM ANALYZE <table_name> (<column1>, <column2>, ...);
```
8. VACUUM for a Specific Schema
```
VACUUM <schema_name>.<table_name>;
```
9. Autovacuum Configuration in postgresql.conf (for background VACUUM)
```
Enable/Disable autovacuum:
autovacuum = on
autovacuum_naptime = 60  # (time in seconds)
autovacuum_vacuum_threshold = 50
autovacuum_analyze_threshold = 50
autovacuum_vacuum_scale_factor = 0.2
autovacuum_analyze_scale_factor = 0.1
vacuum_cost_delay = 20  # (in milliseconds)
vacuum_cost_limit = 200  # (I/O operations before the process pauses)
```
 -------------------------------------------------------------------------------------------------------
- Post-VACUUM Commands:
```
Analyze Table (after VACUUM):
This updates the table statistics for optimized query planning:
ANALYZE <table_name>;
Running VACUUM on Multiple Tables or Databases:
Vacuum all tables in a database:
VACUUM;
Vacuum a specific set of tables:
VACUUM <table_name1>, <table_name2>;
Note on Privileges:
Only the table owner or a superuser can run VACUUM on a table.
By using these syntaxes, you can perform all types of vacuum-related maintenance tasks in PostgreSQL, from basic cleaning to advanced disk space reclamation and wraparound prevention.
```
