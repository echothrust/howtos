---
---

# OpenBSD LDAP Tips
---
author: Pantelis Roditis
date: unknown
modified: 29/12/2022
note: This is quite old and probably heavily outdated

---
- [OpenBSD LDAP Tips](#openbsd-ldap-tips)
  - [OpenBSD ldapd (ldapd.conf)](#openbsd-ldapd-ldapdconf)
  - [Setting up OpenBSD LDAPD(8)](#setting-up-openbsd-ldapd8)
    - [Basic configuration](#basic-configuration)
    - [SSL configuration](#ssl-configuration)
    - [Schemas](#schemas)
    - [ACLS](#acls)
    - [Indexes](#indexes)
    - [Populating the server](#populating-the-server)
  - [Maintenance](#maintenance)
    - [Indexes](#indexes-1)
    - [Backup](#backup)
    - [Restore](#restore)
  - [References](#references)


## OpenBSD ldapd (ldapd.conf)

* First find naming context:
```
ldapsearch -x -h localhost -s base -b "" +
```

* Then browse through the entries in the direcotry:
```
ldapsearch -x -h localhost -b "dc=echothrust,dc=net" -s one
```

* Restrict returned attributes. Only return DNs:
```
ldapsearch -x -h localhost -b "ou=people,dc=echothrust,dc=net" -s one 1.1
```

* Search for all people who work in accounting:
```
ldapsearch -x -h localhost -b "dc=echothrust,dc=net" "(&(objectclass=person)(ou=accounting))"
```

* In order to start feeding our ldap directory with data we need to configure
the root objects first. In our case the top most domain and our default manager
account.
```
dn: dc=echothrust,dc=net
objectclass: dcObject
objectclass: organization
dc: echothrust
o: Echothrust Solutions
#
dn:cn=Manager,dc=echothrust,dc=net
objectclass: organizationalRole
cn: Manager
```

* Import the data by either running `ldapmodify -a` or `ldapadd`
```
ldapmodify -a -x -h ldap.echothrust.dev -D cn=Manager,dc=echothrust,dc=net -w password -f init.ldif
```

## Setting up OpenBSD LDAPD(8)

Structure for the ldap will be as follows
* We will have three namespaces
 * dc=echothrust,dc=com
 * dc=echothrust,dc=net
 * dc=echothrust,dc=org

* dc=echothrust,dc=com will have the following sub entries
 * ou=AddressBook Our addressbook holder
 * ou=Companies this will hold groupofentries to denote employees of a single
 company.

* dc=echothrust,dc=net will have the following sub entries
 * ou=equipment that will hold our asset management
 * ou=users Our system corporate users with authentication details
 * ou=groups our system groups
 * ou=hosts (cn+iphostaddress) list of hosts


### Basic configuration
### SSL configuration
### Schemas
### ACLS
### Indexes
### Populating the server

## Maintenance
Every once in a while (and assuming a lot of changes have taken place on your
ldap tree) it is good to compact and re-index your databases by executing the
following commands respectively

### Indexes
```sh
ldapctl compact
ldapctl index
```

In order to see statistics about the server you can execute
```
ldapctl stats
```

### Backup
You can use slapcat(8) to generate an LDIF file.
```
slapcat -f slapd.conf -b "dc=echothrust,dc=net" -l echothrust.ldif
```

### Restore
The fastest way to load initial data is from an LDIF file. Make sure slapd is not running when you do this.
```
 # kill -9 slapd
 # slapadd -b dc=echothrust,dc=net -l echothrust.ldif -f slapd.conf
```

## References
* http://puffysecurity.com/wiki/ypldap.html
* http://jpmens.net/2010/11/02/ldapd-a-lightweight-ldap-server/
* http://www.tumfatig.net/20120817/monitoring-openbsds-ldap-daemon/
* https://www.tumfatig.net/20110104/back-to-the-sea-the-lightweight-directory-ldap-episode-v/
* http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man5/ldapd.conf.5?query=ldapd.conf
* http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man8/ldapctl.8?query=ldapctl&sec=8
* http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man8/ldapd.8?query=ldapd&sec=8
* Invoice details schema https://github.com/stepping-stone/ldap-schemas/blob/master/sst.schema