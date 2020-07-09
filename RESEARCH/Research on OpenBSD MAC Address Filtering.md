# Research on OpenBSD MAC Address Filtering
**Author:** Pantelis Roditis
**Created:** 2006/08/15 01:11

## Introduction
OpenBSD comes with a plethora of security tools which enable it to operate under extreme conditions and environments. In the following document an attempt for a proficient solution that will provide MAC address filtering with easy incorporation into existing daemons will be examined.

## Permanent ARP Entries
Just add the users to a file (`/etc/users_ip_mac.table`)

  1.2.3.1 00:01:02:03:04:05 permanent
  1.2.3.2 00:0A:0B:0C:0D:0E permanent

Build the static arp table by executing

  /usr/sbin/arp -f /etc/users_ip_mac.table

Next, just block the users. (`/etc/pf.conf`)

  pass in log quick from 1.2.3.0/24 to any label authorized_users

Ofcourse thisn't the only thing one has to do since there is no currently a clear way to distinguish users between them.

## Bridge Configuration Tool
Another way to perform mac filtering is by using a "half" bridge to perform the filtering for you. Unfortunately, there is no solid way to remove and add rules (sort of like `pfctl -f /etc/pf.conf`).

Create a bridge interface and add the desired interface.

  brconfig bridge0 add fxp0

Add tags to the desired mac addresses

  brconfig bridge0 rule pass in on fxp0 src 00:01:02:03:04:05 tag users

And now you can filter like normal through the pfctl interface.

  pass in quick on fxp0 tagged users

Rules can also be stored into their own file for clarity and simplecity. Place
your rules just like the example and load them with the option rulefile.
```
# cat /etc/bridge.d/0_sis0.conf
pass in on sis0 src 00:01:02:03:04:05 tag users
pass in on sis0 src 01:01:02:03:04:05 tag users
# brconfig bridge0 rulefile /etc/bridge.d/0_sis0.conf
```

In order to achieve a similar result with `pfctl -Fa -f /etc/pf.conf` you can
do so by executing the following command.

  brconfig bridge0 flushrule sis0 rulefile /etc/bridge.d/0_sis0.conf


## Using DHCP
Since OpenBSD 4.0 the dhcpd server can be used to add IP addresses that have successfully requested and obtained a lease to a PF table. Combined with the feature of dhcpd that allows one to lease IP addresses only to specific ethernet addresses you can easily perform MAC Filtering.

The idea is to restrict users in using dhcpd leased addresses and block all others. Define the PF tables that dhcpd will use and also block address not coming from these addresses:
  table <dhcp_leased> persist
  table <dhcp_abandoned> persist
  ...
  #allow packets that come from leased addresses block abandoned addresses
  block in all
  block quick from <dhcp_abandoned>
  pass from <dhcp_leased> to any

Then on the dhcpd.conf :
  group {
    host pc1 {
      hardware ethernet 00:01:02:03:04:05;
      fixed-address 10.0.0.50;
    }
  }

And running dhcpd :
  /usr/sbin/dhcpd -A dhcp_abandoned -L dhcp_leased <if>

Where `<if>` is the interface (or interfaces separated by whitespaces) where dhcpd should answer requests on

Once the host with MAC Address 00:01:02:03:04:05 requests an IP using DHCP and if it obtains it successfully then it should be added to the table dhcp_leased. Once the host doesn't renew its lease then dhcpd will add him to dhcpd_abandoned.

Care should be taken since MAC Addresses as well as IP Addresses are easily spoofed using [[http://www.openbsd.org/cgi-bin/man.cgi?query=ifconfig&sektion=8&arch=i386&apropos=0&manpath=OpenBSD+Current|ifconfig(8)]].
