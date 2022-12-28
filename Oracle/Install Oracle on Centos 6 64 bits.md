# Install Oracle on Centos 6 64 bits

base file is 10201_database_linux_x86_64.cpio.gz

## System preparation

Install a X server
```
yum groupinstall "X Window System"
```
Or, for a remote install on a server that doesn't need a graphical interface, install the following packages and dependencies to do X11 forwarding:
```
yum install xorg-x11-xauth.x86_64 xorg-x11-apps.x86_64
```

Change the timezone to GMT
```
# rm /etc/localtime
# ln -s /usr/share/zoneinfo/GMT /etc/localtime
```
Install some additional packages (yes, oracle installer requires GCC.... :/ )
```
# yum install libXp libXtst binutils compat-db compat-libstdc++-33 glibc glibc-devel glibc-headers gcc gcc-c++ libstdc++ cpp make libaio ksh elfutils-libelf sysstat libaio libaio-devel setarch  libXp.i686 libXtst-1.0.99.2-3.el6.i686 glibc-devel.i686 libgcc-4.4.4-13.el6.i686 compat-libstdc++* compat-libf2c* compat-gcc* compat-libgcc* libXt.i686 libXtst.i686
```
On CentOS 6 64 bits, I also found the glibci686 to be necessary, otherwise /lib/ld-linux.so.2 is missing:
```
yum install glibc-2.12-1.7.el6_0.5.i686
```
Create user oracle and group oinstall
```
# groupadd oinstall
# groupadd dba
# useradd -d /home/oracle -g oinstall -G dba -s /bin/bash oracle
# passwd oracle
Changing password for user oracle.
New UNIX password:
Retype new UNIX password:
```
Add limits to /etc/security/limits.conf
```
# grep oracle /etc/security/limits.conf
# specific to oracle
oracle    soft  nproc  2047
oracle    hard  nproc  16384
oracle    soft  nofile  1024
oracle    hard  nofile  65536
```
For limits to be applied on login in /etc/pam.d/login
```
# grep limit /etc/pam.d/login
session required /lib64/security/pam_limits.so
```
## oracle sysctl parameters
Modify the SYSCTL parameters in **/etc/sysctl.conf**

```
# max number of file descriptors
fs.file-max = 65535

# maximum size in bytes of a single shared memory segment
# that a Linux process can allocate in its virtual address space.
# with 64GB of RAM, we set the max segment size at 60GB, thus the
# SGA cannot be larger than 60GB for one database instance
# This is 1GB=1073741824 Multiply with the GB of memory to set the
# correct value
kernel.shmmax = 1073741824

# total amount of shared memory pages that can be used system wide.
# Hence, SHMALL should always be at least ceil(shmmax/PAGE_SIZE).
# with 60GB of RAM and a page size at 4KB
kernel.shmall = 15728640

# system wide maximum number of shared memory segments
kernel.shmmni = 4096

# control the number of semaphores on the system
# kernel.sem = semmsl semmns semopm semmni
#
# SEMMSL kernel parameter is used to control the maximum number
# of semaphores per semaphore set.
# SEMMSL setting should be 10 plus the largest PROCESSES
# parameter of any Oracle database on the system, here 2048
#
# SEMMNS parameter is the maximum number of semaphores that
# can be allocated (SEMMSL * SEMMNI) system wide
#
# SEMOPM kernel parameter is used to control the number of
# semaphore operations that can be performed per semop system call
#
# SEMMNI kernel parameter is used to control the maximum number
# of semaphore sets on the entire Linux system.
kernel.sem = 2048 262144 2048 128

# usuable port range
net.ipv4.ip_local_port_range = 9000 65500

# default OS receive buffer size in bytes
net.core.rmem_default = 262144

# max OS receive buffer size in bytes
net.core.rmem_max = 4194304

# default OS transmit buffer size in bytes
net.core.wmem_default = 262144

# max OS transmit buffer size in bytes
net.core.wmem_max = 1048576
```
Make sure the hostname is known in /etc/hosts, otherwise the TNS listener will refuse to start
```
$ grep alp7 /etc/hosts
127.0.0.1       alp7.domain.com      alp7
```
And maybe remove SELINUX
```
# vim /etc/sysconfig/selinux
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#       enforcing - SELinux security policy is enforced.
#       permissive - SELinux prints warnings instead of enforcing.
#       disabled - SELinux is fully disabled.
SELINUX=disabled
# SELINUXTYPE= type of policy in use. Possible values are:
#       targeted - Only targeted network daemons are protected.
#       strict - Full SELinux protection.
SELINUXTYPE=targeted

# SETLOCALDEFS= Check local definition changes
SETLOCALDEFS=0
```
Followed by a reboot.

## Oracle installation

Log in as oracle and extract the installer
```
$ gunzip 10201_database_linux_x86_64.cpio.gz
$ cat 10201_database_linux_x86_64.cpio |cpio -idmv
```
Then, run it :
```
$ ./runInstaller -ignoreSysPrereqs
```
Run the scripts as root (asked by the installer):
```
# /opt/oracle/oraInventory/orainstRoot.sh
Changing permissions of /opt/oracle/oraInventory to 770.
Changing groupname of /opt/oracle/oraInventory to oinstall.
The execution of the script is complete
```
```
# /opt/oracle/oracle/product/10.2.0/db_1/root.sh
Running Oracle10 root.sh script...

The following environment variables are set as:
    ORACLE_OWNER= oracle
    ORACLE_HOME=  /opt/oracle/oracle/product/10.2.0/db_1

Enter the full pathname of the local bin directory: [/usr/local/bin]:
   Copying dbhome to /usr/local/bin ...
   Copying oraenv to /usr/local/bin ...
   Copying coraenv to /usr/local/bin ...


Creating /etc/oratab file...
Entries will be added to the /etc/oratab file as needed by
Database Configuration Assistant when a database is created
Finished running generic part of root.sh script.
Now product-specific root actions will be performed.
```
The script created /etc/oratab. Oratab must be populated with a list of databases to start automatically
```
$ cat /etc/oratab
#



# This file is used by ORACLE utilities.  It is created by root.sh
# and updated by the Database Configuration Assistant when creating
# a database.

# A colon, ':', is used as the field terminator.  A new line terminates
# the entry.  Lines beginning with a pound sign, '#', are comments.
#
# Entries are of the form:
#   $ORACLE_SID:$ORACLE_HOME:<N|Y>:
#
# The first and second fields are the system identifier and home
# directory of the database respectively.  The third filed indicates
# to the dbstart utility that the database should , "Y", or should not,
# "N", be brought up at system boot time.
#
# Multiple entries with the same $ORACLE_SID are not allowed.
#
#
ALP1:/opt/oracle/oracle/product/10.2.0/db_1:Y
IMP:/opt/oracle/oracle/product/10.2.0/db_1:Y
EMAIL:/opt/oracle/oracle/product/10.2.0/db_1:Y
```
Add ENV variables to /opt/oracle/.bashrc
```
export ORACLE_HOME=/opt/oracle/oracle/product/10.2.0/db_1
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME/lib
export PATH=$ORACLE_HOME/bin:$PATH
export ORACLE_SID=orcl
```
Log into SQL plus (for fun)
```
$ sqlplus / as sysdba

SQL*Plus: Release 10.2.0.1.0 - Production on Tue Apr 19 17:46:08 2011

Copyright (c) 1982, 2005, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> quit

```
Patch Oracle using patchset to 10.2.0.4. Launch the ./runInstaller and follow the instructions. Make sure to select to proper Oracle Home (the existing one, not a new one).

Create a startup script in /etc/init.d/dbora
```
#!/bin/sh -e

# chkconfig: 3 56 10
# description: Oracle 10G custom start/stop script

DAEMON=oracle
ORACLE_HOME=/opt/oracle/oracle/product/10.2.0/db_1
ORACLE_OWNER=oracle

restart() {
    stop
    start
}

case $1 in
    'start')
        su - ${ORACLE_OWNER} -c "${ORACLE_HOME}/bin/lsnrctl start"
        su - ${ORACLE_OWNER} -c "${ORACLE_HOME}/bin/dbstart"
        su - ${ORACLE_OWNER} -c "${ORACLE_HOME}/bin/emctl start dbconsole"
        su - ${ORACLE_OWNER} -c "${ORACLE_HOME}/bin/isqlplusctl start"
    ;;
    'stop')
        su - ${ORACLE_OWNER} -c "${ORACLE_HOME}/bin/isqlplusctl stop"
        su - ${ORACLE_OWNER} -c "${ORACLE_HOME}/bin/emctl stop dbconsole"
        su - ${ORACLE_OWNER} -c "${ORACLE_HOME}/bin/dbshut"
        su - ${ORACLE_OWNER} -c "${ORACLE_HOME}/bin/lsnrctl stop"
    ;;
    restart)
        restart
    ;;
    *)
        echo "Usage: $0 {start|stop}"
        exit
    ;;
esac

exit $?

```

Start Oracle:
```
(as root)
# /etc/init.d/dbora start
```
Add dbora to runlevel 3
```
# chkconfig --add dbora
```


### Create a database=

We want to create a database called **WH1** (ORACLE_SID=WH1).

#### Create an init file

Create the file in **ORACLE_HOME/dbs/initWH1.ora** with the following content.

```
control_files = (/home/oradata/wh1/WH1/control01.ctl,
                /home/oradata/wh1/WH1/control02.ctl,
                /home/oradata/wh1/WH1/control03.ctl)
undo_management =       auto
db_name =               WH1
db_block_size =         8192
audit_file_dest =       /home/oradata/wh1/admin/adump
background_dump_dest =  /home/oradata/wh1/admin/bdump
core_dump_dest =        /home/oradata/wh1/admin/cdump
db_domain =             example.com
db_recovery_file_dest = /home/oradata/wh1/flash_recovery_area
db_recovery_file_dest_size = 42949672960
db_unique_name =        WH1
user_dump_dest =        /home/oradata/wh1/admin/udump
sga_max_size =          4294967296
sga_target =            4294967296
pga_aggregate_target =  1073741824
```

#### Create the folders
Make sure that all the folders listed in the init file are created on the system:

```
$ mkdir -p /home/oradata/wh1/{WH1,admin,archivelogs,flash_recovery_area}
$ mkdir -p /home/oradata/wh1/admin/{a,b,c,u}dump
```

#### Initialize the datase

This is done in sqlplus, first we export the env variable ORACLE_SID, then we start the oracle instance processes and finally we create the database with a custom query that specify the location of the datafiles, etc...

```
$ export ORACLE_SID=WH1
$ env|grep ORACLE_SID
ORACLE_SID=WH1
```

Now connect to the idle instance
```
$ sqlplus / as sysdba

SQL*Plus: Release 10.2.0.4.0 - Production on Wed Oct 12 15:38:51 2011

Copyright (c) 1982, 2007, Oracle.  All Rights Reserved.

Connected to an idle instance.

SYS@WH1 AS SYSDBA>
```

Start the process but don't mount anything (nothing exists yet).

```
SYS@WH1 AS SYSDBA> startup nomount;
ORACLE instance started.

Total System Global Area 4294967296 bytes
Fixed Size                  2187776 bytes
Variable Size             815570432 bytes
Database Buffers         3472883712 bytes
Redo Buffers                4325376 bytes
```

This create a bunch of processes on the system:
```
$ ps -edf|grep WH1

  oracle  8106     1   0 15:42:53 ?           0:00 ora_smon_WH1
  oracle  8100     1   0 15:42:52 ?           0:00 ora_dbw1_WH1
  oracle  8102     1   0 15:42:52 ?           0:00 ora_lgwr_WH1
  oracle  8110     1   0 15:42:53 ?           0:00 ora_mmon_WH1
  oracle  8092     1   0 15:42:52 ?           0:00 ora_pmon_WH1
  oracle  8098     1   0 15:42:52 ?           0:00 ora_dbw0_WH1
  oracle  8096     1   0 15:42:52 ?           0:01 ora_mman_WH1
  oracle  8104     1   0 15:42:52 ?           0:00 ora_ckpt_WH1
  oracle  8113  8089   0 15:42:53 ?           0:00 oracleWH1 (DESCRIPTION=(LOCAL=YES)(ADDRESS=(PROTOCOL=beq)))
  oracle  8094     1   0 15:42:52 ?           0:00 ora_psp0_WH1
  oracle  8112     1   0 15:42:53 ?           0:00 ora_mmnl_WH1
  oracle  8108     1   0 15:42:53 ?           0:00 ora_reco_WH1
```

We can create the database with the following SQL query

```
create database WH1
  user sys identified by "123456"
  user system identified by "123456"
  logfile   group 1 ('/home/oradata/wh1/WH1/redo1.log') size 100M,
            group 2 ('/home/oradata/wh1/WH1/redo2.log') size 100M,
            group 3 ('/home/oradata/wh1/WH1/redo3.log') size 100M
  datafile '/home/oradata/wh1/WH1/system01.dbf'
                size 500M
                autoextend on
                next 100M maxsize unlimited
                extent management local
  sysaux datafile '/home/oradata/wh1/WH1/sysaux01.dbf'
            size 100M
            autoextend on
            next 10M
            maxsize unlimited
  undo tablespace undo
            datafile '/home/oradata/wh1/WH1/undotbs01.dbf'
            size 100M
            autoextend on
            next 10M
            maxsize unlimited
  default temporary tablespace temp
            tempfile '/home/oradata/wh1/WH1/temp01.dbf'
            size 100M
            autoextend on
            next 10M
            maxsize unlimited
  default tablespace warehouse
      datafile '/home/oradata/wh1/WH1/wh01.dbf'
                size 500M
                autoextend on
                next 100M maxsize unlimited
                extent management local;
```

```
SYS@WH1 AS SYSDBA> create database WH1
  user sys identified by "123456"
  user system identified by "123456"
 [....]

Database created.

SYS@WH1 AS SYSDBA>
```

The following queries display the created files:

```
SYS@WH1 AS SYSDBA> select name from v$datafile;

NAME
--------------------------------------------------------------------------------
/home/oradata/wh1/WH1/system01.dbf
/home/oradata/wh1/WH1/undotbs01.dbf
/home/oradata/wh1/WH1/sysaux01.dbf
/home/oradata/wh1/WH1/wh01.dbf

SYS@WH1 AS SYSDBA> select member from v$logfile;

MEMBER
--------------------------------------------------------------------------------
/home/oradata/wh1/WH1/redo1.log
/home/oradata/wh1/WH1/redo2.log
/home/oradata/wh1/WH1/redo3.log

SYS@WH1 AS SYSDBA> select name from v$controlfile;

NAME
--------------------------------------------------------------------------------
/home/oradata/wh1/WH1/control01.ctl
/home/oradata/wh1/WH1/control02.ctl
/home/oradata/wh1/WH1/control03.ctl

```

We can, right away, add 2 more datafiles to this tablespace:

```
SYS@WH1 AS SYSDBA> alter tablespace warehouse add datafile
     '/home/oradata/wh1/WH1/wh02.dbf' size 500M
     autoextend on next 100M maxsize unlimited;

Tablespace altered.

SYS@WH1 AS SYSDBA> alter tablespace warehouse add datafile
     '/home/oradata/wh1/WH1/wh03.dbf' size 500M
     autoextend on next 100M maxsize unlimited;

Tablespace altered.
```
#### Create the Catalog and PL/SQL structure

The following oracle scripts are required to finish the creation of the database:

```
SQL> @?/rdbms/admin/catalog.sql
SQL> @?/rdbms/admin/catproc.sql
```
The shortcut **@?** represents ORACLE_HOME. It could be replace with the value of $(echo $ORACLE_HOME).

catalog.sql creates the data dictionary. catproc.sql creates all structures required for PL/SQL.

#### Grant permissions

We create 3 users : a DBA, a writer and a reader. They all work in the namespace of the DBA.

The Master DBA:
```
SYS@WH1 AS SYSDBA> create user wh_master identified by "123456" default tablespace warehouse;

User created.

SYS@WH1 AS SYSDBA> grant DBA to wh_master;

Grant succeeded.
```


The Reader:
```
SYS@WH1 AS SYSDBA> create user wh_reader identified by "whGLN$2011" default tablespace warehouse;

User created.
```

## Database restoration

Recreate folder structure
```
# mkdir /home/oradata
# chown oracle:oinstall /home/oradata/
# mount --bind /mnt/oradata/ /home/oradata/
# ls /home/oradata/
backup_area
```
Restore previous database
```
$ set ORACLE_SID=ALP1
$ rman target / nocatalog
> startup nomount force;
> set dbid 2893065500;
> restore spfile from '/path/to/last/autobackup';
```
DBID is the ID of the database that can be found in the control file name (eg. c-<DBID>-<DATE>-<number> such as c-2893065500-20110426-01).

Note; The restore spfile will fail if the database already has an spfile. In this case, reuse or move the previous spfile.

```
$ sqlplus / as sysdba
> create pfile = '/tmp/pfile.txt' from spfile;
```
Check /tmp/pfile.txt and make sure all paths mentionned in the file exists.

```
mkdir -p /opt/oracle/admin/ALP1/{a,b,c,u}dump
mkdir -p /home/oradata/alp1/ALP1/
mkdir -p /opt/oracle/db.ctrl/ALP1/
mkdir -p /home/oradata/alp1/flash_recovery_area
mkdir -p /home/oradata/alp1/archivelogs
```

Now start oracle with the new spfile.
```
SQL> shutdown abort;
SQL> startup nomount;
```
Alter memory usage and restart once again
```
SQL> alter system set sga_target = 2G scope=spfile;
SQL> alter system set sga_max_size = 2G scope=spfile;
SQL> alter system set pga_aggregate_target = 500M scope=spfile;
SQL> shutdown abort;
SQL> startup nomount;
```
To make the changes on the system running, change the scope to 'memory' (or use 'both' directly). DO NOT decrease sga_target while running (kswapd will go crazy and system will freeze eventually). You can increase sga_target up to sga_max_size and Oracle will dynamically increase the memory gradually.

Restore the datafiles
```
# su oracle
$ rman target / nocatalog
RMAN> restore controlfile from '/home/oradata/backup_area/autobackup/c-2893065500-20110328-01';
RMAN> alter database mount;
RMAN> restore database;
RMAN> recover database;
```
The recovery will finish with an error because it cannot find the archive log. This is normal, we need to reset the archive log.
```
RMAN> alter database open resetlogs;
```
For a backup list, inside RMAN:
```
> crosscheck backup; // Will check availability of files on disk and set their status (available or expired)
> list backup summary; // Will show brief backup list - note ID of last FULL backup
> list backup; // Lots of output - need to be sure that all files in FULL backup and after it are available
```

Copy password file from original source
```
# cp orapwALP1 /opt/oracle/oracle/product/10.2.0/db_1/dbs/
# chown oracle:oinstall orapwALP1
```
Or recreate a new one
```
$ orapwd file=/opt/oracle/oracle/product/10.2.0/db_1/dbs/orapwALP1 password=123456 entries=5
```


Restart and mount the database:
```
SQL> shutdown abort;
ORACLE instance shut down.

SQL> startup mount;
ORACLE instance started.

Total System Global Area 2147483648 bytes
Fixed Size          2085360 bytes
Variable Size         486542864 bytes
Database Buffers     1644167168 bytes
Redo Buffers           14688256 bytes
Database mounted.

SQL> alter database open;

Database altered.

```
### Install Enterprise Manager=

Configure new EMCA (database needs to be open)
```
$ emca -deconfig dbcontrol db
$ emca -config dbcontrol db -repos recreate
```

If this method doesn't work, drop the existing repository manually.
```
$ sqlplus / AS sysdba

SQL> conn sysman/sysman
Connected.

SQL> exec DBMS_AQADM.DROP_QUEUE_TABLE(queue_table=>'MGMT_NOTIFY_QTABLE',force =>TRUE);
PL/SQL procedure successfully completed.

$ sqlplus / AS sysdba

SQL> DROP user sysman cascade;
DROP role MGMT_USER;
DROP user MGMT_VIEW cascade;
DROP public synonym MGMT_TARGET_BLACKOUTS;
DROP public synonym SETEMVIEWUSERCONTEXT;
User dropped.
SQL>
Role dropped.
SQL>
Synonym dropped.
SQL>
Synonym dropped.


SQL> DECLARE
CURSOR c1 IS
SELECT owner, synonym_name name
FROM dba_synonyms
WHERE table_owner = 'SYSMAN';
BEGIN
FOR r1 IN c1 LOOP
IF r1.owner = 'PUBLIC' THEN
EXECUTE IMMEDIATE 'DROP PUBLIC SYNONYM '||r1.name;
ELSE
EXECUTE IMMEDIATE 'DROP SYNONYM '||r1.owner||'.'||r1.name;
END IF;
END LOOP;
END;
/
PL/SQL procedure successfully completed.
System altered.
```

Then, create the repository without dropping the old one:
```
$ emca -config dbcontrol db -repos CREATE
```

### Oratab=

Copy oratab from original source
If the destination is solaris, oratab must be in /var/opt/oracle/. Otherwise, it must be in /etc/.
```
# cp /var/opt/oracle/oratab /etc/
```

System related:
local dircolor: /root/.dir_colors with dir in yellow (33) instead of blue (34)
change default runlevel in /etc/inittab (from 5 to 3)

#### Rsync backup from a host to another

To preserver the directory structure and transfert only the most recent backups, use the following steps:
1. create a list of the file modified during the last 2 days
```
# find /home/oradata/backup_area -mtime -2 -type f > /tmp/backuplist
```
2. create (touch) the file on the target machine, make sure the directory structure exists
```
# for i in $(cat /tmp/backuplist); do touch $i;done
```
3. on the source machine, use rsync to transfert only the file existing on the target machine
The rsync command makes sure that the modification time is not taken into account when building the transfert list (only the file size). bwlimit ensures that we don't consume more than 7MB of bandwidth.
```
rsync --partial --progress --bwlimit=7000 --existing --size-only -av -e "ssh -i /opt/oracle/.ssh/id_dsa" /home/oradata/backup_area oracle@184.72.111.143:/home/oradata/
```



### Migration Procedure=

Note to migrate a running database to a new server with a little downtime as possible. The procedure assume that both servers share a common location to store the backup files. Server A will perform a FULL hot backup, then shut down and perform a cold incremental backup. Server B will then do a restore of the entire backup (FULL + INCR) and restart with a clean database.


### Server A: Full Hot Backup followed by Cold Incremental=
On server A
```
$ sudo su - oracle

$ export ORACLE_SID=ALP1

# rotate the online redo log

$ sqlplus / as sysdba << EOF
ALTER SYSTEM SWITCH LOGFILE;
EOF

# perform the full backup

$ rman target / nocatalog << EOF
BACKUP INCREMENTAL LEVEL 0 CUMULATIVE DATABASE TAG 'FULL';
EOF

# rotate the online redo log

$ sqlplus / as sysdba << EOF
ALTER SYSTEM SWITCH LOGFILE;
EOF

# perform the archive logs backup

$ rman target / nocatalog << EOF
BACKUP ARCHIVELOG ALL NOT BACKED UP DELETE ALL INPUT TAG 'ARCH';
EOF
```

The hot backup finished, we turn off all applications accessing the database and make sure that all connections to 1521 are closed.
The shutdown command below will commit all non-committed transactions and do a clean shutdown. But it will hang if there are still active sessions that are not closed.

```
# shut down the database

$ sqlplus / as sysdba << EOF
SHUTDOWN TRANSACTIONAL;
STARTUP MOUNT;
EOF

# perform the cold incremental backup

$ rman target / nocatalog << EOF
BACKUP INCREMENTAL LEVEL 1 CUMULATIVE DATABASE TAG 'COLDINCR' INCLUDE CURRENT CONTROLFILE;
EOF

$ rman target / nocatalog << EOF
BACKUP ARCHIVELOG ALL NOT BACKED UP DELETE ALL INPUT TAG 'ARCH';
EOF
```

#### Server B: Full restore and recovery of the database

```
$ sudo su - oracle

$ export ORACLE_SID=ALP1

$ sqlplus / as sysdba << EOF
SHUTDOWN TRANSACTIONAL;
STARTUP NOMOUNT;
EOF

# restore the latest controlfile, and then the database
$ rman target / nocatalog
RMAN> restore controlfile from '/home/oradata/backup_area/autobackup/<LATESTCONTROLFILE>';
RMAN> alter database mount;
RMAN> restore database;
RMAN> recover database;
```

<note warning>
**Relocating backups**

If a backup location changes, RMAN will not find the backup files. To recatalog the files, use the command "catalog"
```
    RMAN> catalog start with '/mnt/backup-rman/backup/';
```
</note>
The database can not be reopened
```
$ sqlplus / as sysdba

> alter database open resetlogs;
```

tips: make sure that the SCN are the same on both database
```
> select to_char(current_scn) from v$database;
```


====== Install desktop environment after CentOS minimal installation ======
<code sh>
yum install @basic-desktop
yum groupinstall "X Window System" "Desktop Platform"
</code>

====== Install Development Platform after CentOS ======
<code sh>
yum install kernel-devel
yum groupinstall "Development Tools"
</code>
{{tag>howto oracle preparation centos}}