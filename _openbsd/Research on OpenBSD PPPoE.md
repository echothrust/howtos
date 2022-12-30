---
---

# OpenBSD PPPoE
This was an research paper written by one of our members back in 2006. This
research was made in order to familiarize with the available technologies under
OpenBSD.

**Author:** Panagiotis Efstratiou
**Created:** 2006/07/06 02:05


## Introduction
### What is PPP ?
PPP stands for Point-to-Point Protocol. It is a protocol designed for directly connecting two (2) peers. This is done by several means, that is the protocol has the ability to be adopted on several Layer2 technologies. It also has the ability to adopt and incorporate any Layer3 protocol (that includes IP and IPX, though using PPP for IPX is becoming increasingly rare). Finally PPP incorporates several authentication methods for the peers.

### What is PPPoE ?
PPPoE stands for PPP over Ethernet. As told in the above section PPP can be used over several Layer2 Technologies one of which is Ethernet. For this to work both a kernel driver and a user-land program are required.

## OpenBSD and PPPoE
OpenBSD has support for both PPP and PPPoE. There exist both drivers and user-land programs (that use to `tun` interface). Though the kernel drivers are much faster but have less options. Therefore it is recommended that we use only the user-land programs. No extra kernel configuration is required in order to achieve this since OpenBSD ships with the required options enabled by default.

## The PPP Program
 The PPP program ([[http://www.openbsd.org/cgi-bin/man.cgi?query=ppp&apropos=0&sektion=8&manpath=OpenBSD+Current&arch=i386&format=html|ppp(8)]]) can be used both as a client and a server.
 It supports the most used technologies including :

  * Various Authentication Techniques :
    * PAP
    * CHAP
    * MS-CHAP
    * MS-CHAPv2
  * It can be used for encaptulation Over
    * Serial TTYs
    * IP using TCP or UDP
    * Ethernet
  * It supports IPCP Negotiation
  * It supports NAT
  * It supports Packet Filtering
  <sub>(For a complete list of feature please refer to the manual page)</sub>

## The PPPoE Program
The PPPoE program ([[http://www.openbsd.org/cgi-bin/man.cgi?query=pppoe&apropos=0&sektion=8&manpath=OpenBSD+Current&arch=i386&format=html|pppoe(8)]]) work in conjection with the PPP program. It can act both as a server (thus listening for incoming PPPoE Requests) and as client (connecting/sending PPPoE requests to other PPPoE servers).



## PPP Program Configuration
PPP configuration is divided into sections which are called 'systems'. Each
system has a unique label and the options below it are applied only to the
connection specified. There is a global section that can be used for setting
system wide this section is called 'default'.

PPP stores its configuration to the following files :
  * /etc/ppp/ppp.conf - Main configuration file
  * /etc/ppp/ppp.linup - Configuration file that is parsed when a link goes up
  * /etc/ppp/ppp.linkdown - Configuration file that is parsed when a link goes down

## Sample client configuration
To create a client PPPoE configuration just create a system in the main configuration file (look above) as follows :
```
pppoe:
  set device "!/usr/sbin/pppoe -i <interface>"
  set mtu max 1492
  set mru max 1492
  set speed sync
  disable acfcomp protocomp
  deny acfcomp
  set authname "<myUsername>"
  set authkey "<myPassword>"
```
Where interface is an Ethernet that is only up and running (there is no need to setup an IP address for that interface) and myUsername and myPassword are the credentials used for authenticating to the peer. You may also notice a smaller than 1500 mtu and mru size, this is done to give PPP enough space for its headers.

The final step is to dial this system using the ppp(8) command <sub>(Please refer to the manual page for dialing mode)</sub>.

It will automatically invoke the pppoe(8) program. This will bind to standard input and standard output and will allow ppp(8) to handle the PPP protocol while pppoe(8) will handle to the Ethernet layer.

## Sample server configuration
 As you have noticed in the above example for pppoe(8) to act as a client we first invoke ppp(8) and then pppoe(8) is invoked as a child process. The exact opposite is done when we want to use pppoe(8) as server. We have to invoke pppoe(8) in such a way that it will listen for incoming PPPoE Requests and the invoke ppp(8) to handle the PPP Session.

To do so we should invoke pppoe(8) as follows :
   pppoe -i <interface> -s -p <system> [-n <service>]

 The `interface` has the same semantics as the `interface` in '3.2' the `system` value is the ppp(8) system to use when invoking the program. The `service` value can be a PPPoE Service that is sent along with client's credentials. It is an optional value.

 Now we have to setup the specified `system` in our ppp(8) main configuration file. This section should contain information about how the peer is to authenticate itself (that is with that what protocol) and what range of IP addresses can use and should be assigned to it. For a speed-up hack we will also proxy all ARP requests from the client to the server. So far by this version of ppp(8) routes are added automatically. If `system` is set to pppoe-in you can use the following configuration :

           pppoe-in:
              set mtu max 1492
              set mru max 1492
              set speed sync
              disable acfcomp protocomp
              deny acfcomp
              enable pap
              enable chap
              enable MSCHAP
              enable MSCHAPv2
              set timeout 120
              set ifaddr 10.0.1.1 10.0.1.2-10.0.1.120
              enable proxy

 The above configuration allows the peer to authenticate itself using PAP, CHAP, MSCHAP or MSCHAPv2 (look at section 4 for a complete explanation of authentication techniques that can be used with ppp(8)) and allows them to negotiate through IPCP an IP address from withing the range of 10.0.1.2 to 10.0.1.120.

## Authentication through PPP
### Authentication protocols supported
  * PAP - PAP stands for Password Authentication Protocol. The main idea of the protocol is to send the username and the password of the client so many until we get a successful connection. Username and Password are sent in plain-text thus this protocol is insecure.
  * CHAP - CHAP stands for Challenge Authentication Protocol. When a client chooses CHAP to authenticate itself it firstly sends its username to the server, then the server looks-up its username/password database to find the corresponding password. It then encrypts this password with an one-way hash algorithm and sends over to the client. The client encrypts its password and matches it with the one of the server. If this matching is successful the connection is established.
  * MSCHAP - MSCHAP is a variation of CHAP that was developed by MicroSoft. It uses a hash algorithm that is a combination of DES and MD4. It is not very secure since it has small keys thus MicroSoft developed MSCHAPv2.
  * MSCHAPv2 - MSCHAPv2 is an upgraded version of MSCHAP it uses bigger keys thus making it more secure.

 The problem that arises from the various Protocols that are supported is that different clients use different protocols. For example most UNIXes tend to use PAP or CHAP to authenticate themselves while newer MicroSoft clients (that is Windows98SE and on) tend to prefer MSCHAPv2. Other older MicroSoft clients will MSCHAP. It is mandatory for the server to support all these protocols.

### Authentication back-ends supported
 ppp(8) can support three different back-ends.
  * ppp.secrets - This is a plain-text file which has a specific format that contains username/password information for each user as well as other information that can be used from ppp(8).
  * passwd(5) - Authentication is done using the system's password database.
  * RADIUS - RADIUS stands for Remote Access Dial-in Service. ppp(8) can query a RADIUS server (and a backup server) for username/password credentials as well as other settings (such as the IP addresses to be used for IPCP negotiation).

 passwd(5) backend cannot be used for CHAP or MSCHAP[v2] authentication. That is because CHAP has to know the plain-text password. Also the standard way of using back-ends is to first search within ppp.secrets then in passwd(5) and then to query RADIUS.

## Appendix A - ppp.secrets format
The ppp.secrets file uses the following format
```
  <username><TAB><password><TAB><IP Address><TAB><Label><TAB><Callback>
```

Since Callback is a protocol beyond the scope of this document we will not discuss the <Callback> option here.
   * `username` is the username that the peer has to use for authentication.
   * `password` is the password that the peer has to use to authenticate itself. It can have the special value '*' (asterisk). If this is the case then the passwd(5) mechanism will be used (note that if the client has choosed a protocol for authenticating other the PAP the connection will fail).
   * `IP Address` is the IP address to assign to the client. This can be used for accounting purposes. This field may contain a range of IP addresses specified either by CIDR notation or by simple IP ranges (as in the **set ifaddr** option described above) it may also have the special value of '*' (asterisk) which means that ppp(8) will assign a random IP address from the list that was specified by the **set ifaddr** option.
   * `Label` is the section within ppp.linkup and ppp.linkdown files that will be parsed when this user connects and disconnects. It may be empty or have the value '*' (asterisk) in which case it will be ignored.
