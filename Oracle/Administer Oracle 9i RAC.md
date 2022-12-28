# Administer Oracle 9i RAC
**Need to know**:

  * Database Name
  * Instance Name
  * pfile location
  * Storage Method (ASM/Filesystem)


**Use SRVCTL to verify database/instance**:
* Obtain the status of GSD `gsdctl status`
* First run the GSD and listener
```
%ORACLE_HOME%\bin\gsdctl start
%ORACLE_HOME%\bin\lsnrctl start
```
* List configuration of the instance & database:
```
srvctl config instance
srvctl config database
srvctl config database -d dbname
```
* To list all environment variables for a database: `srvctl getenv database -d mydb`
* List DB and instance status
```
srvctl status database -d database_name
srvctl status instance -d mydb -i mydb1,mydb2
```
* start/stop db
```
srvctl start database -d mydb (-i iname - optionally)
srvctl stop database -d mydb
```

**IF everything fails try**

Start `OracleServicesid` instance on each node. `C:\> net start OracleServicesid`

From the Control Panel's Services window, select `OracleServicesid`, then click `Start`.

If the listener is not started, start it on each of the nodes
```
LSNRCTL
LSNRCTL> start [listener_name]
```

Where `listener_name` is the name of the listener defined in the `listener.ora` file.

You do not have to identify the listener if you are using the default listener named LISTENER.
```
sqlplus /nolog
CONNECT SYS\password
STARTUP PFILE=%ORACLE_HOME%\database\initsid.ora;
```

## Export the database

Once the database is open, ensure that EXP_FULL_DATABASE and is assigned to DBA role
```sql
SELECT * FROM DBA_SYS_PRIVS;
SELECT * FROM DBA_ROLE_PRIVS;
```

If the role does not exist then run `catproc.sql` and `catalog.sql`

Ensure the SYSTEM user has the DBA role assigned
  C:\>EXP system/manager FULL=y FILE=export.dmp

SOS: IF stored procedures exists, they cannot be migrated to MySQL!!!!
```sql
SELECT * FROM ALL_OBJECTS WHERE OBJECT_TYPE IN ('FUNCTION','PROCEDURE','PACKAGE');
```