# Research on OpenBSD bridge(4)
## 1. Introduction
### 1.1 What is bridging?
Bridging is the ability to transform one Layer 2 Packet from one type to
another. For example with bridging we can transform an IEEE 802.11a/b/g (WiFi)
packet to an IEEE 802.3 (IEEE Ethernet Compatible Extension). Bridges are also
used for connecting two or more LANs together.

### 1.2 How do they work ?
Bridges simply forward a packet from one interface to all other available
interfaces that participate in the bridge configuration. Much like a switch it
has the ability to learn which hosts belong to which interface thus minimizing
the costs for packet delivery. This is done by intercepting ARP packets on
these interfaces. Multiple bridges can be used to connect to a single LAN and
the bridge can detect and disable the loops using a Spanning Tree algorithm
incorporated with a Spanning Tree Protocol (either simple STP or RSTP may be
used - see below).

Finally, bridges used to comfort to the least MTU, when bridging interfaces
with different MTUs, to avoid inconsistencies, since Layer 2 does not support
fragmentation. This is no longer done as most bridges use a store-and-forward
technique and therefore allow fragmentation at Layer 3.

### 1.3  What is remote bridging?
Bridges can be established on local LANs (such as between two offices in the
same building or between WiFi and Ethernet) but also on remote LANs as well
(e.g. between two offices in different cities). This technique is called Remote
Bridging and is usually achieved by means of Layer 3 tunnels between the two
end-points. Those tunnels are specifically configured to allow higher Layer 2
frames (referred to as Ethernet Frames from know on) to pass through them.

Note that Layer 3 fragmentation cannot be used over such a tunnel since it
can't fragment the Ethernet Frames any more.

## 2. OpenBSD Bridges
### 2.1 OpenBSD bridge(4) Interface
OpenBSD, like most modern Operating Systems (not including Mac OS X), has
support for bridges through the [[http://tinyurl.com/42qqw6|bridge(4)]] driver.
This driver allows the normal behavior of bridge (as described above) and also
supports remote bridging through the generic tunneling interface
[[http://tinyurl.com/5b68oa|gif(4)]]. To start using the bridging capabilities
of OpenBSD one should first execute the following command :
```
ifconfig bridgeX create
```

where `X` is a number. This command will create the bridge interface that can be
later used with the [[http://tinyurl.com/c6ork|brconfig(8)]] command.

### 2.2 brconfig(8) command
The [[http://tinyurl.com/c6ork|brconfig(8)]] is used to set and get all
settings on a bridge interface. It can be used to add or delete an interface or
to allow STP over an interface. We demonstrate the most important uses of this
command.

The following command will add interfaces `xl0` and `re0` to `bridge0`
```
brconfig bridge0 add xl0 add re0
```

The following command will enable STP (or RSTP support) on interface `xl0`
```
brconfig bridge0 stp xl0
```

The following command will change the Spanning Tree Protocol from Rapid STP to
simple STP (NOTE : OpenBSD supports both and enables RSTP by default - see below)
```
brconfig bridge0 proto stp
```

The following command will allow `bridge0` to pass
[[http://tinyurl.com/5kmrd|IPSec(4)]] traffic over it and it will instruct the
[[http://tinyurl.com/6kpcdk|isakmpd(8)]] to setup any SAs as necessary
```
brconfig bridge0 link2
```

### 2.3 Spanning Trees
As mentioned before, a bridge uses Spanning Tree algorithms to detect and
disable loops. Bridges use special Ethernet Frames to communicate with each
other.

During this communication they elect a spanning tree root, usually the bridge
who's interface has the smallest designated number (this is identical to the
MAC Address of the interface). Then they begin constructing a spanning tree
using the costs of each link, eventually this will detect double links and
choose to disable the link with the biggest cost. This is the normal behavior
of STP (Spanning Tree Protocol) and the behavior of the enhanced predecessor
STP, RSTP (Rapid Spanning Tree Protocol). RSTP is better than STP in a sense
that addresses several problems such the equal costs links and administrators
are encouraged to use this instead of STP.

## 3. OpenBSD Remote Bridges
### 3.1 Tunnel Interfaces
So to create a remote bridge one should first create a bridge then a tunnel to
connect the two (or more) endpoints together. In OpenBSD only
[[http://tinyurl.com/5b68oa|gif(4)]] allows the forwarding of Ethernet Frames.

It does so using a specific protocol name EtherIP which is specified in
[[http://tinyurl.com/4b5dm9|RFC 3378]]. So let us consider that we have two
OpenBSD routers, A and B, which both have an `re0` interface connected to their
LANs and an `xl0` interface which is connected to the internet. `xl0` on router
A has an IP Address of `1.2.3.4` and `xl0` on router B an IP address of `4.3.2.1`.

First we have to configure the tunnel interfaces between A and B.
* On A we write
```
ifconfig gif0 create
ifconfig gif0 tunnel 1.2.3.4 4.3.2.1
ifconfig gif0 up
```

* On B we write
```
ifconfig gif0 create
ifconfig gif0 tunnel 4.3.2.1
ifconfig gif0 up
```

These commands will create a tunnel between A and B. Notice we did not assign
any IP Address to the tunnel interfaces. It is actually useless to do so since
the interfaces will forward Ethernet Frames only (Layer 2 Packets).

### 3.2 sysctl(8) Settings
For now we have our tunnels setup but they are not ready to forward EtherIP
traffic. OpenBSD will silently drop that traffic by default. So to change that
behavior a [[http://tinyurl.com/5wbv6b|sysctl(8)]] options exists by the name
of `net.inet.etherip.allow`. This option is off by default so we issue the
following command
```
sysctl net.inet.etherip.allow=1
```

### 3.3 Tying it all together
So after one has created the tunnels he should issue the following commands on
both A and B to create and bring the bridge up:
```
ifconfig bridge0 create
brconfig add re0 add gif0
brconfig up
```

Now traffic will be forwarded from LAN A to LAN B as if they where connected by
normal bridges.

To configure the bridge and tunnel interfaces automatically at startup you
should use the [[http://tinyurl.com/44r5xo|hostname.if(5)]] mechanism.

## 4. Troubleshooting and Caveats
First of all as we mentioned above no Layer 3 fragmentation is possible at
Ethernet Frames forwarded through [[http://tinyurl.com/5b68oa|gif(4)]] tunnels
so be aware to set the MSS of TCP connections (maybe using
[[http://tinyurl.com/yv5av7|pf(4)]] scrubbing rules) below 1280 (the lowest
MTU for a [[http://tinyurl.com/5b68oa|gif(4)]] interface).
Many people should notice by now that we forward plain Ethernet Frames over
insecure networks (this can be done over the internet for example). Refer to
Appendix A on how to setup [[http://tinyurl.com/5kmrd|ipsec(4)]] over the
[[http://tinyurl.com/5b68oa|gif(4)]] interface.

To troubleshoot the bridge use the [[http://tinyurl.com/c6ork|brconfig(8)]]
with arguments `-A` or `-a` to see all the configured bridges as well as all
learned MAC Addresses on all interfaces. Make sure that the correct addresses
are learned on the correct interfaces and so on.

## Appendix A - IPSec(4) over Remote Bridges
If one has some experience on using and configuring [[http://tinyurl.com/5kmrd|ipsec(4)]]
you would notice by now that the easiest way to protect our Ethernet Frames is
to create an SA that will encrypt the EtherIP protocol (which travels over the
[[http://tinyurl.com/5b68oa|gif(4)]] interfaces). That SA should be in
transport mode and not in tunnel mode.

To do so we use the [[http://tinyurl.com/yjvoqf|ipsecctl(8)]] command with a
suitable [[http://tinyurl.com/68zxbl|ipsec.conf(5)]] file after starting
[[http://tinyurl.com/6kpcdk|isakmpd(8)]]. We will not focus on how to create
SAs on OpenBSD since it off the scope of this document but we will show you a
sample [[http://tinyurl.com/68zxbl|ipsec.conf(5)]] file

* In /etc/ipsec.conf on host A
```
ike dynamic esp transport proto etherip from 1.2.3.4 to 4.3.2.1 \
  main auth hmac-sha2-512 enc blowfish quick auth hmac-ripemd160 enc aesctr \
  srcid 1.2.3.4 dstid 4.3.2.1
```

* In /etc/ipsec.conf on host B
```
ike dynamic esp transport proto etherip from 4.3.21 to 1.2.3.4 \
  main auth hmac-sha2-512 enc blowfish quick auth hmac-ripemd160 enc aesctr \
  srcid 4.3.2.1 dstid 1.2.3.4
```

Then on both hosts:
```
ipsecctl -f /etc/ipsec.conf -vvv
```

## References
[[http://tinyurl.com/42qqw6|bridge(4)]]
[[http://tinyurl.com/5b68oa|gif(4)]]
[[http://tinyurl.com/c6ork|brconfig(8)]]
[[http://tinyurl.com/5kmrd|ipsec(4)]]
[[http://tinyurl.com/6kpcdk|isakmpd(8)]]
[[http://tinyurl.com/4b5dm9|RFC 3378]]
[[http://tinyurl.com/5wbv6b|sysctl(8)]]
[[http://tinyurl.com/44r5xo|hostname.if(5)]]
[[http://tinyurl.com/yv5av7|pf(4)]]
[[http://tinyurl.com/yjvoqf|ipsecctl(8)]]
[[http://tinyurl.com/68zxbl|ipsec.conf(5)]]
