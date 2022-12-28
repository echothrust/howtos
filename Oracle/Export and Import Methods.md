# Export and Import Methods

## Export schema and import to a new tablespace on the same DB
You cannot move the whole schema from one tablespace to another, especially if you have LOBs. What you need to do is either transfer only the data (without the LOBs), which is too much of a trouble and you may loose some information unless you are extremely careful. Or you can export the schema you want to move to another tablespace, delete the objects of that schema and import the objects to new tablespace. In order to export a schema and import it to the same database but to new tablespace you need to:

* **Create the new tablespace(s)** with the appropriate datafiles. We are going to create a test tablespace with starting with 10G of space and it will be auto-extendable. The auto-extend will occur every 250M and the maximum size will be unlimited.
```
sql> create tablespace tablespace_name datafile 'path_to_file/filename.dbf'
   > size 10G autoextend on next 250m maxsize unlimited extent management local;
```
* Create a directory on the filesystem that will hold the dump files of export/import procedure.
```
$ mkdir /path_to_dir/exp_dir
```
* Register the directory to the database and give permissions to a db user in order to be able to export/import objects.
```
sql> create or replace directory new_imp_dir as '/path_to_dir/exp_dir';
sql> grant read,write on directory new_imp_dir to system;
```
* Check the number and the status of schema objects
```
sql> select object_type,status,count(*) from user_objects group by object_type,status;
```
* Export the schema (user) you wish.
```
$ expdp system/password directory=new_imp_dir dumpfile=name_of_dump_file.dmp logfile=name_of_log_file.log schemas=name_of_schema
```
* Delete the schema objects.
