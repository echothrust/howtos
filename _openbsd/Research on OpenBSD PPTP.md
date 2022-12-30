---
---

# Research on OpenBSD PPTP
## 1. Introduction


### 1.1 What is PPTP ?
**PPTP** stands for `Point-to-Point Tunneling Protocol`. It was originally
created by Microsoft as tunneling protocol over PPP. It was later adopted by
IETF as [[http://www.faqs.org/rfcs/rfc2637.html|RFC 2637]].


### 1.2 How PPTP Works ?
All that PPTP does is to start a negotiation using TCP/IP at port 1723 then
negotiate the creation of a PPP tunnel over TCP/IP. All authentication is done
through PPP which also negotiates an IP and potential routes with the peer
using IPCP.

## 2. PPTP and UNIX

### 2.1 PoPToP
So it is obvious that a user-land daemon is required to start the negotiation then a PPP implementation can be used to allow the user to authenticate. That user-land daemon is called [[http://www.poptop.org|PoPToP]] which was originally created for Linux (what a surprise) but it also exists on *BSD ports (and of course OpenBSD's ports) so what one should do, is :
```
cd /usr/ports/net/poptop
make build
make package
make install
make clean
```

This will easily download, patch and install PoPToP for OpenBSD.
Then you need a nice configuration. By default the configuration file of pptpd is located at `/etc/pptpd.conf` so edit this file as follows :
```
option /etc/ppp/ppp.conf
localip 192.168.1.233
remoteip 192.168.1.234-238
listen 10.10.10.10
pidfile /var/run/pptpd.pid
```

  * option : your ppp(8) options file.
  * localip : This is the IP used for our tunnel end-point and should be an unused IP on your local network
  * remoteip : The range of remote IPs that the peer will be assigned to.
  * listen : the remote IP which has access to the internet. PoPToP will bind to this IP and listen on TCP port 1723 (see above).
  * pidfile : It's far from obvious.


### 2.2 PPP
There's a catch up here. OpenBSD (and other BSD flavours) has done very little to implement PPP on the kernel-side so a user-land program is used instead [[http://www.openbsd.org/cgi-bin/man.cgi?query=ppp&apropos=0&sektion=8&manpath=OpenBSD+Current&arch=i386&format=html|ppp(8)]].
This program is feature-full and has much of the compatibility needed by Microsoft Windows to connect to a PPTP-based VPN. You need to setup a `ppp.conf` which is located at `/etc/ppp/ppp.conf`. A sample configuration should look like this :
```
default:
        set log lcp ipcp
loop:
        set timeout 0
        set device localhost:pptp
        set dial
        set login
        set mppe * stateful
        set ifaddr 192.168.1.233 192.168.1.234-192.168.1.238 255.255.255.0
        set server /tmp/loop "" 0177
loop-in:
        set timeout 0
        allow mode direct
        set log phase lcp ipcp command
pptp:
        load loop
        set dns 192.168.1.1
        disable pap
        enable chap
        enable MSCHAP
        enable MSCHAPv2
        set nbns 192.168.1.2
        enable proxy
        allow proxy
        allow proxyall
        enable proxyall
        disable ipv6
        accept mppe
        set device !/etc/ppp/secure
```

Let's start to examine each section :
  - pptp : This section is the default section that is called by the PoPToP Daemon
      * load loop : this will load the loop section which contains some startup settings that will be negotiated with the client using the TCP port (more on that later)
      * Then we disable PAP and enable CHAP, MSCHAP and MSCHAPv2. MSCHAP and MSCHAPv2 is the default used by Microsoft Windows while PAP is considered quite insecure for such use.
      * set nbns : Tells the peer which IP should use for NetBIOS Lookups
      * The next 4 lines allow the peers to exchange ARP packets thus allowing hosts on both sides to communicate directly.
      * We disable ipv6 since it's not yet widely deployed but it can be used. Note that disabled options are not negotiated at all.
      * We accept mppe, allowing the user to use `Microsoft's Point-to-Point Encryption` scheme which allows several modes of encryption. ppp(8) fully supports all of them.
      * Then reset the device back to a script which actually calls the `loop-in` label to create a loop for the tun interface. At this point PoPToP will stop caring about the `tunX` interface created by ppp(8).
   - loop : This section does the first negotiation
      * We set the device to be the port opened by PoPToP and set some settings for negotiation.
      * set login : Defines that the user should be authenticated using at least one mechanism (by default by searching inside `/etc/ppp/ppp.secrets`)
      * set mppe * stateful : Encrypts the connection using MPPE and allows the client to use any encryption key size.
      * set ifaddr 192.168.1.233 192.168.1.234-192.168.1.238 255.255.255.0 : Sets the IP addresses and netmasks that are to be negotiated with the peers using IPCP. These should be the same as the ones you defined in `/etc/pptpd.conf`.

Now the shell script that we use :
`/etc/ppp/secure`
```
#!/bin/sh
exec /usr/sbin/ppp -direct loop-in
```


### 2.3 Tying it all together
First set the executable flag on `/etc/ppp/secure` :
```
chmod 700 /etc/ppp/secure
```

Then add a user at `/etc/ppp/ppp.secret` :
```
# username      password        IP      label   callback
test         testing         *
```

Then run the PoPToP Daemon :
```
/usr/local/sbin/pptpd
```

If no errors were displayed then you are good to go. Just try the connection using a Microsoft Windows client.
