---
---

# OpenBSD unbound and nsd mini howto
The scope of this how to is to show a simple configuration of an
OpenBSD server as NSD (Authoritative) +Unbound (Recursive) name server.

Since it is a 2-leg installation in order to add a zone on Authoritative name
server the procedure is the following.

  - Create the zone file with the appropriate content
  - Add the new zone to `nsd.conf`
  - Run `nsdc rebuild && nsdc reload`
  - Add the new zone to unbound.conf
  - Restart unbound

In order to have a BIND's split horizon on NSD/Unbound setup we must deploy multiple NSD instances

## NSD part
The configuration of NSD is located under `/etc/nsd.conf`. For R&D we create a new subdomain (i.e. lab2.echothrust.dev) and its reverse zone.

```
# $OpenBSD: nsd.conf,v 1.5 2011/12/15 14:51:46 sobrado Exp $
server:
    hide-version: yes
    database: "/var/nsd/db/nsd.db"
    username: _nsd
    logfile: "/var/log/nsd.log"
    pidfile: "/var/nsd/run/nsd.pid"
    difffile: "/var/nsd/run/ixfr.db"
    xfrdfile: "/var/nsd/run/xfrd.state"
    ip-address: "10.0.2.254"
    port: "5353"

## master zone example
zone:
    name: "lab1.echothrust.dev"
    zonefile: "lab1.echothrust.dev"

zone:
    name: "1.0.10.in-addr.arpa"
    zonefile: "10.0.1"
```

Since we are going to host both services on the same host we change the default
port (53) of NSD in order to avoid conflicts with `unbound`.

NSD keeps a journal file with the contents of the zone transfers
(difffile: "ixfr.db" setting in the configuration file). This file
will slowly grow as zone transfers are done. To clean up the journal
file issue a patch command `nsdc patch`

## Unbound
The configuration files of Unbound are located under `/var/unbound/etc`. In
order to know which ROOT-SERVER should ask for non-Authoritative domains we
must download the `root.hints` from ftp://FTP.INTERNIC.NET/domain/named.cache.

In order to be able to use the `unbound-control` for start/stop/restart the
daemon, we must create certs and keys for unbound. To accomplish that we run
`unbound-control-setup` and unbound places 2 pairs of certs and keys.
One for the server side and one for the client. The default location of the
pairs is under `/var/unbound/etc/` directory.

```
#
# Example configuration file.
#
# See unbound.conf(5) man page, version 1.4.17.
#
# this is a comment.

#Use this to include other text into the file.
#include: "otherfile.conf"

# The server clause sets the main parameters.
server:
  verbosity: 1
  hide-version: yes
  # We haven't applied any trust-anchors in order to use 'validator' so
  # we use iterator
  module-config: "iterator"

  # We need the following rules in order to let nsd handle the reverse lookup for our lan address space.
  # Seems that unbound.conf can't accept subnets of the 10/8-network (i.e 2.0.10.in-addr.arpa.).
  local-zone: "10.in-addr.arpa." nodefault
  domain-insecure: "10.in-addr.arpa"

  interface: 10.0.1.254
  interface: 10.1.1.254
  interface: 127.0.0.1

  access-control: 172.16.10.0/24 deny
  access-control: 127.0.0.0/8 allow
  access-control: 10.0.1.0/24 allow
  access-control: 10.1.1.0/24 allow

  cache-max-ttl: 10
  do-ip6: no
  chroot: "/var/unbound"
  root-hints: "/etc/root.hints"
  do-ip6: no

remote-control:
  control-enable: yes
  server-key-file: "unbound_server.key"
  server-cert-file: "unbound_server.pem"
  control-key-file: "unbound_control.key"
  control-cert-file: "unbound_control.pem"


forward-zone:
  name: "."
  forward-addr: 172.16.10.254

stub-zone:
  name: "lab1.echothrust.dev"
  stub-addr: 10.0.1.254@5353

stub-zone:
  name: "dmz1.echothrust.dev"
  stub-addr: 10.0.1.254@5353

stub-zone:
  name: "1.0.10.in-addr.arpa"
  stub-addr: 10.0.1.254@5353
```
