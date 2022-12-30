---
---

# Troubleshoot ORA-00600
Basic steps to be followed while solving ORA-00600 [4194]/[4193] errors. Information in this document applies to any platform.

* Oracle Database - Enterprise Edition - Version 7.0.16.0 and later.
* Checked for relevance on 03-Oct-2011
* Oracle Server Enterprise Edition

## Short Description of ORA-00600[4194],[a],[b]
A mismatch has been detected between Redo records and rollback (Undo) records.

ARGUMENTS:
  * Arg [a] Maximum Undo record number in Undo block
  * Arg [b] Undo record number from Redo block


## Option 1: Supported Method (Drop the undo tablespace).
**Single instance**
This error normally happens for a new transaction. The trace file actually shows an active transaction for the undo segment because this is the transaction created by the same process.If the undo segment happens to have an active transaction , then Oracle may recover it later with no problems .

Normally if the header is dumped after the error, the active transaction is gone.

So a simpler option to resolve this issue is.

* **Step 1**
  ```sql
  SQL> Startup nomount ;    --> using spfile
  SQL> Create pfile='/tmp/corrupt.ora' from spfile ;
  SQL> Shutdown immediate;
  ```
* **Step 2** \
  Modify the corrupt `.ora` file and set `Undo_management` to `Manual`
  ```sql
  SQL> Startup mount pfile='/tmp/corrupt.ora'
  SQL> Show parameter undo
  ```
  it should show manual
  ```sql
  SQL> Alter database open ;
  ```
  If the database comes up
  ```sql
  SQL> Create rollback segment r01 ;
  SQL> Alter rollback segment r01 online ;
  ```
  Create a new undo tablespace
  ```sql
  SQL> Create undo tablespace undotbs_new datafile '<>' size <> M ;
  ```

* **Step 3**
  ```sql
  SQL> Shutdown immediate;
  SQL> Startup nomount ; ---> Using spfile
  SQL>Alter system set undo_tablespace=<new Undo tablespace created> scope=spfile;
  SQL> Shutdown immediate ;
  ```

* **Step 4** \
Drop the old undotbs tablespace.
```sql
SQL> Drop tablespace <undo tablespace name> including contents and datafiles
```
Please note: You can delay the drop of the Old undo tablespace this is just to allow the block clean out to happen for dead transactions.

So the below step can be issued after database has been up and running with new undo tablespace for couple of hours.

Also note if your database has been forced open (datafiles are not in sync and archivelogs missing) using any unsupported method then please do not drop the Old undo.

## Option 2: For RAC Instance (when one instance is down and the other is running)
If one node is up and running and other node is failing with ORA-00600[4194]/[4193] then

From the instance which is up and running create a new undo tablespace and make it the default one for the other instance which is down with the error.Startup the failing instance with
the new undo tablespace.

From Instance which is up and running
```sql
Create undo tablespace undo_new datafile '<filename>' size <> m ;
Alter system set undo_tablespace=<New undo tablespace name> sid=<instance which has corrupt undo tablespace and is down> scope=spfile ;
```

Now Startup the Instance which is down
```sql
SQL>Startup mount
SQL>Show parameter undo
```

Should show the new undo tablespace created above
```sql
SQL>Alter database open ;
SQL>Drop tablespace <Old undo tablespace of the failing instance> including contents and datafiles
```

If all the Instances are down in the RAC due to this error then following the instruction given for Single instance and creating new undo tablespace for each should solve it.
