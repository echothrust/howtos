---
---

# OpenBSD relayd SSL acceleration and load-balancing
NOTE: Taken from misc but cant find the original post

We have 3x Supermicro Intel Dual Xeon E5-2620v3 powered systems with 32GB ECC
memory, 4x 10 Gigabit Ethernet NICs (Intel X520-DA2), and 2x Gigabit Ethernet
onboard NICs connected towards a Virtual Chassis of a Juniper EX 4550 Ethernet
Switch, running OpenBSD 5.8 with all (11) patches.

We want to use these 3 systems as loadbalancers, 2x 10GE (trunk0, LACP)
inbound, 2x 10GE (trunk1, LACP) outbound, 2x 1GE (trunk2, LACP) for Pfsync.

LB-1 shares a public IP with LB-2, and LB-2 and LB-3 do the same (via CARP). We
use relayd for Loadbalancing the traffic towards 3 backend servers, all they
currently do is serving a HTTP 200 OK response.

When we load tested one LB's HTTP performance alone with wrk - we get about
40k req/s when testing with one machine in the same network as a client, and
more than 100k req/s when testing with 3 client machines. Doing the test with
HTTPS brings the performance down to 1400 req/s, and it does not matter if
using more or less clients, the total number of req/s stays almost the same.

The overall load of the systems is low (below 2-3), memory utilization is
low as well.

As we don't have experience with OpenBSD and relayd we can only compare
these numbers to FreeBSD and HAproxy, which we used in our previous setup. Our
configuration files are listed below - we would be happy about any comment
how to improve the HTTPS performance.

by optimizing and simplifying pf.conf rules and relayd.conf we were able to
push 24400 req/s through with HTTPS.

## sysctl.conf
```
net.inet.carp.preempt=1
kern.bufcachepercent=90
kern.maxfiles=200000
kern.maxproc=50000
kern.maxclusters=32768
net.inet.ip.forwarding=1
net.inet6.ip6.forwarding=1
net.inet.ip.ifq.maxlen=8192
net.inet.ip.mtudisc=0
net.inet.tcp.rfc3390=1
net.inet.tcp.mssdflt=1440
```

## relayd.conf
```
ip4_244 = "xx.xx.xx.244"
ip4_245 = "xx.xx.xx.245"

tracker5 = "10.5.3.34"
tracker6 = "10.5.3.42"
tracker7 = "10.5.3.50"

interval 10
table <trackers> { $tracker5, $tracker6, $tracker7 }

prefork 12

http protocol https {

  ### TCP performance options
  tcp { nodelay, sack, socket buffer 65536, backlog 128 }

  match request header append "X-Forwarded-For" value "$REMOTE_ADDR"
  match request header append "X-Forwarded-By" \
      value "$SERVER_ADDR:$SERVER_PORT"
  match header set "Keep-Alive" value "$TIMEOUT"

  pass
  tls { no tlsv1.0, ciphers "HIGH:!aNULL" }
  tls session cache disable

}

relay wwwssl {
  listen on $ip4_244 port 443 tls
  listen on $ip4_245 port 443 tls
  protocol "https"
  forward to <trackers> port 8083 mode roundrobin check tcp
  session timeout 60
}

relay www {
  listen on $ip4_244 port 80
  listen on $ip4_245 port 80
  forward to <trackers> port 8083 mode roundrobin check tcp
}

```

## pf.conf
```
tcp_services = "{ domain }"
udp_services = "{ domain }"
tcp_public_services = "{ www, https }"

pfsync_int = trunk2 # Pfsync interface
int_if = trunk1 # DMZ (internal) interface
ext_if = trunk0 # External CARP interface

# Increase limits
set limit { states 25000, src-nodes 25000, table-entries 300000 }

# Aggressive settings
set optimization aggressive
set timeout { adaptive.end 120000, interval 2, tcp.tsdiff 5, tcp.first 5,
tcp.closing 5, tcp.closed 5, tcp.finwait 5, tcp.established 4200}

# See pf.conf(5) and /etc/examples/pf.conf
anchor "relayd/*"

set block-policy drop
set loginterface $ext_if
set skip on lo
set skip on $int_if
set skip on $pfsync_int

match in all scrub (no-df max-mss 1440)

# Block everything by default
block all

# Allow main service of this host
pass quick proto tcp to port $tcp_public_services keep state
pass out quick proto tcp to port $tcp_services keep state
pass proto udp to port $udp_services keep state

# Pass CARP
pass quick proto carp keep state (no-sync)

# SSH backup channel from Wooga office
pass in on trunk0 inet proto tcp from xx.xx.xx.xx/xx to any port 22 keep
state (no-sync)

# Allow pings for Pingdom status checks
pass on trunk0 inet proto icmp keep state (no-sync)
pass on trunk0 inet6 proto icmp6 keep state (no-sync)
```