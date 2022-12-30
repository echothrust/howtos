---
---

# OpenBSD YP Server Setup
* **Author:** Pantelis Roditis
* **Created:**  2006/08/14 03:50

## Introduction
Configuring YP server under OpenBSD. This guide is old, outdated and insecure. You have been ___warned___.

## Prepare the system
First we have to prepare the system so that the services start whenever the server restarts.
Edit /etc/rc.conf.local and set the following settings.
```sh
      portmap=YES
      ypserv_flags=
      yppasswdd_flags=    # Allows password changes from remote machines
```

## Enable NIS master server
Set the NIS domain and start the daemon so that we can start working on our NIS whithout restarting.

```sh
    echo "mynisdomain" > /etc/defaultdomain
    domainname mynisdomain
    ypinit -m
```

## Preparing Directory Services
In order to avoid security mishabs we create a fake directory that mimics the /etc folder files that the YP server needs. This way we have different users populated through our YP server than the local system. We try to ensure that the server risks as litle as possible.

```sh
    mkdir /etc/fakeyp
    cp /etc/{group,hosts,ethers,networks,rpc,services,protocols,netid,netgroup,aliases} /etc/fakeyp
    egrep "(user1|user2|user3)" /etc/master.passwd > /etc/fakeyp/master.passwd
```

Edit the files accordingly. A good thing to do is remove all passwords from the master.passwd located under /etc/fakeyp. (also remember to remove the accounts from the local database /etc/master.passwd as it will conflict/get overwriten). Furthermore, remember to keep in mind the UID/GID numbers as it is easier to mess with them when not using the system utils.

Once done simply create the derived passwd file from the master.passwd by executing
```sh
   cd  /etc/fakeyp
   cap_mkdb -d /etc/fakeyp -p /etc/fakeyp/master.passwd
```

This will create all the required files needed for NIS.

## Building YP Databases
Now to /var/yp/mynisdomain/ and edit the Makefile switching the appropriate variables.
```
# Directory to use default '/etc'
# Change this to the fakeyp location
DIR=/etc/fakeyp
# We dont care about encrypted prouts prouts since we dont use passwords at all.
# We simply want a way to distribute UID/GID pairs.
UNSECURE="True"
# Look dns for unknown hosts seems like a handy option default empty
# Now that depending on your network topology this might leak unecessary information to outsiders.
# in our case we have a filtering DNS server that prohibits certain queries from even reaching the root servers.
USEDNS=-b
```

## Start the daemons
```
portmap
ypserv
ypbind
rpc.yppasswdd
```
These will all be restarted on reboot.

## Testing the services
Test your installation by performing

```
rpcinfo -p
ypcat hosts
id user1
```

## Updating YP databases
If at any time you would like to add a user to the fakeyp/master.passwd you'd have to update the NIS server by simply performing the following steps.

```
cd /var/yp/mynisdomain && make
```

This will fetch the updated versions and push them into the clients no need to restart the daemons for this operation.
