# OpenBSD YP Client Setup
* **Author:** Pantelis Roditis
* **Created:** 2006/08/14 03:49

## Introduction
Configuring an OpenBSD to act as YP client and join a YP Domain.

## Preparing the system
Edit `/etc/rc.conf.local` and add the following lines in order to enable the rpc/nis damons during system startup.
```sh
portmap=YES
amd=YES
rpc.lockd=YES
```

Add the NIS domain on `/etc/defaultdomain`
```sh
echo mynisdomain > /etc/defaultdomain
```

Configure amd to auto mount home directories.

```sh
 echo "/home amd.home" >> /etc/amd/master
```

**Note** that the automounted home is mounted on top of the existing /home.

## Append NIS maps to passwd and group files
Add the following entry `+:*::::::::` into your password databases using `vipw`
```sh
pwd_mkdb /etc/master.passwd
echo "+:*::" >> /etc/group
```
## Start the daemons
```
portmap
ypbind
amd -x error,noinfo,nostats -a /tmp_mnt -l syslog /home amd.home
rpc.lockd
```

These will be started automaticaly after reboot.

## Testing and Maintenance
The above configuration should allow users with accounts created on the master server to login on the client machines without creating new accounts and to have a familiar shell environment. Passwords can be changed from client machines using `yppasswd`.

Things to check if it's not working:
```
    rpcinfo -p host      # to see rpc registered progs on host
    mount -v             # to check nfs mounts
    ypcat amd.home       # to check yp amd.home map
    ypmatch user passwd  # to check user entry in passwd map
    id                   # to check user belongs to correct groups
    less /var/log/daemon # error messages
    amq                  # various options for controlling amd
```
