# Install Oracle n CentOS Linux

## Pre-installation Tasks =====
In order to install Oracle you need to meet some minimum hardware requirements and prepare the linux kernel a bit (by creating users and groups).

## Hardware Requirements
In order to install Oracle Database 10gR2 on CentOS, your system needs to have the following requirements:

  * 512 MB of RAM
  * 1 GB swap space (or twice the size of RAM)
  * 400 MB of disk space in the /tmp directory
  * 1.7 GB, at least, of disk space for the Oracle software. If you want the Sample Schema Database you need at least 2.1G of disk space.

## Configuring Linux
You need to configure the system and create some groups and the oracle user. In order to create the groups you need to:

  * Open a terminal and login as root
  * Create at least the dba and oinstall group. Other groups that can be created are the oper, asmdba, asmoper, dbsnmp
  ```
  # groupadd oinstall
  # groupadd dba
  ```
  * Create the OS user: **oracle**
  ```
  # useradd -g oinstall -G dba oracle
  ```
  * Enter a password for the oracle user you just created
  ```
  # passwd oracle
  ```
  * Change the process and file limits for oracle user by adding the following lines to /etc/security/limits.conf
  ```
  oracle  soft    nproc   2047
  oracle  hard    nproc   16384
  oracle  soft    nofile  1024
  oracle  soft    nofile  65535
  ```
  * Add the following lines to /etc/pam.d/login
  ```
  session    required     /lib/security/pam_limits.so
  session    required     pam_limits.so
  ```
  * Change the user limits for Oracle user by adding the following lines to /etc/profile
  ```
  if [ $USER = "oracle" ]; then
    if [ $SHELL = "/bin/ksh" ]; then
         ulimit -p 16384
    else
         ulimit -u 16384
    fi
  fi
  ```
  * Create the directories needed for the installation and change file ownerships to oracle user
  ```
  # mkdir -p /u01/app/oracle
  # chown -R oracle:oinstall /u01/app
  # chmod -R 775 /u01/app
  ```
  * Set the kernel parameters. Open the **/etc/sysctl.conf** file and **add** lines similar to the following values. These are the minimum values required by oracle, which means that if you have set up already those parameters higher than the values below, you need to skip that step and the one that follows.
  ```
  kernel.sem = 250 32000 100 128
  kernel.shmall = 2097152
  kernel.shmmax = 2147483648
  kernel.shmmni = 4096
  fs.file-max = 65536
  net.core.rmem_default  = 262144
  net.core.rmem_max      = 262144
  net.core.wmem_default  = 262144
  net.core.wmem_max      = 262144
  net.ipv4.ip_local_port_range = 1024 65000
  ```
  * In order for the changes in sysctl to take effect, you need to reboot either the system or to run:
  ```
  # sysctl -p
  ```

## Installing Oracle Database 10.2.1.0
  * As root run the following command:
  ```
  # xhost +
  ```
  * As Oracle user, go to the directory that you have downloaded the oracle software and run the installer.
  ```
  $ ./runInstaller
  ```
