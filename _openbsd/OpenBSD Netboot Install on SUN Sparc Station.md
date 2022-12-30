---
---

# TITLE:OpenBSD Netboot Install on SUN Sparc Station 5/10/20
**Author:** Pantelis Roditis
**Created:** 2006/08/13 20:54

## Introduction
The following document outlines how one can make an OpenBSD net installation for a Sun Sparc Station 5/10/20.

## Scenario Assumptions
The following instructions assume the following
  * **Server IP:** 10.0.0.1
  * **Client name:** ss5
  * **Client MAC:** 01:02:03:04:05:06
  * **Client IP:** 10.0.0.2

## Preparing the Server

* Add the clients MAC address in /etc/ethers with a hostname eg. `ss5`.
```
echo "01:02:03:04:05:06 ss5" >> /etc/ethers
```

* Add an entry in /etc/hosts for the hostname (eg `ss5`) with an IP address.
```
echo "10.0.0.2 ss5" >> /etc/hosts
```

* Start rarpd(8).
```
rarpd -la
```

* Copy boot.net from the OpenBSD/sparc release to `/tftboot/ip-adress.SUN4M)`.
```
cp boot.net $(printf "%.2X" 10 0 0 2).SUN4M
```

* Uncomment the tftp entry in /etc/inetd.conf and restart inetd.
```
kill -HUP $(cat /var/run/inetd.pid)
```

* Create /etc/bootparams containing this:
```
ss5  root=10.0.0.1:/export/ss5/root swap=10.0.0.1:/export/ss5/swap
```

* Start rpc.bootparamd(8).
```
rpc.bootparamd -s
```

* Copy bsd.rd to /export/ss5/root/ and make bsd a symlink to it.
```
ln -s bsd.rd bsd
```

* Edit /etc/exports and add entry for /export:
```
/export -alldirs -network 10.0.0 -mask 255.255.255.0
```

* Create the swap file like this:
```
dd if=/dev/zero of=swap bs=1k count=16000
```

## References
  * diskless(8)
  * rarpd(8)
  * ethers(5)
  * tftpd(8)
  * nfsd(8)
  * exports(5)
  * rpc.bootparamd(8)
  * bootparams(5)
