# LDAP tips for OpenBSD servers
NOTE: This is quite old and may be heavily outdated :)

## OpenLDAP (slapd.conf)

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
ldapmodify -a -x -h ldap.echothrust.com -D cn=Manager,dc=echothrust,dc=net -w password -f init.ldif
```

## Backup

You can use slapcat(8) to generate an LDIF file.
```
slapcat -f slapd.conf -b "dc=echothrust,dc=net" -l echothrust.ldif
```

## Restore
The fastest way to load initial data is from an LDIF file. Make sure slapd is not running when you do this.
```
 # kill -9 slapd
 # slapadd -b dc=echothrust,dc=net -l echothrust.ldif -f slapd.conf
```
