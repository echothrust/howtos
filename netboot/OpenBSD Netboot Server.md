# TITLE:OpenBSD Netboot Server
**Author:** Pantelis Roditis
**Created:** 2007/04/01 17:38

## Introduction
Netbooting is an exciting idea that has its roots way back to the past. However
with the appearance of PXE boot on more devices brought back the subject and
yet again netbooting is back on the map.

Although netbooting in it self is considerably easy, managing and setting up
netboot clients is time consuming and many times requires multiple daemon
restarts.

In the following document we try to outline a simple way for the setting up an
OpenBSD netboot server.

Building an OpenBSD netboot server capable of booting a variety of operating
systems.

## Server Daemon Preparations

### NFS Daemons

#### /etc/bootparams (rpc.bootparamd)
**NOTE** rpc.bootparamd must be run with the command line argument -r 172.16.3.254 (replacing the 172.16.3.254 with your gateway).
```
netvista        root=172.16.3.1:/n/1/root \
                swap=172.16.3.1:/n/1/swap
mantos          root=172.16.3.1:/n/2/root \
                swap=172.16.3.1:/n/2/swap
```

#### /etc/exports
```
  /usr -maproot=0:0 -network=172.16.3 -mask=255.255.255.0
  /home -maproot=0:0 -network=172.16.3 -mask=255.255.255.0
  /n -alldirs -maproot=0:0 -network=172.16.3 -mask=255.255.255.0
```
### DHCP Daemon
```
shared-network LOCAL-NET {
  option  domain-name "rdg.echothrust.org";
  option  domain-name-servers 172.16.1.1;
  option subnet-mask 255.255.255.0;
  option broadcast-address 172.16.3.255;
 # allow bootp;
  allow unknown-clients;
  subnet 172.16.3.0 netmask 255.255.255.0 {
    option routers 172.16.3.254;
    next-server 172.16.3.1;
    server-name "172.16.3.1";
    group {
      filename "pxelinux.0";
      default-lease-time 86400;
      max-lease-time 90000;
      host mantos {
        fixed-address 172.16.3.2;
        hardware ethernet 00:0b:cd:a6:3c:5d;
      }
    }
    group {
      filename "/n/4/root/kernel.2426";
      #filename "/n/1/root/kernel.2200";
      option root-path "172.16.3.1:/n/4/root";
      host netvista {
        fixed-address 172.16.3.102;
        hardware ethernet 00:60:94:2A:50:86;
      }
    }
  }

}
```
### TFTP Daemon
Edit /etc/inetd.conf and add a line like the following
```
tftp            dgram   udp     wait    root    /usr/libexec/tftpd      tftpd -s /tftpboot
```

### RARP Daemon


#### /etc/hosts
```
172.16.3.2 mantos
172.16.3.102 netvista
```

#### /etc/ethers
```
0:b:cd:a6:3c:5d mantos
0:60:94:2A:50:86 netvista
```

## Client specific notes

### IBM NetVista 8363 EXX
This is a tricky machinery that required multiple hacks in order to bring into a usable state. However we are still in the process
of delivering OpenBSD to this thing.

* `M/T Model`: **8363-EXX**
* `Manufacturer`: **IBM**
* `Processor Type`: **Cyrix**
* `Processor Speed`: **300 Mhz**
* `Front Side Bus`: **66Mhz**
* `Cache Size`: **128 K** (?)
* `RAM`: **153 MB** (?)
* `RAM Technology`: **SDRam**
* `Hard Drive Capacity`: **16 MB Card** (?)
* `Graphics Memory`: **2 MB**
* `Sound`: **On Board**
* `Ethernet Support`: **10/100 mbps - On Board**
* `External 3.5 Bays`: **0 Bays External 5.25 Bays: 0 Bays**
* `USB Ports`: **Qty (2)**
* `15 Pin - SVGA Port`: **YES**
* `Power supply`: **12V AC adapter**

{{http://www.bluetrait.com/images/N2200/8363b.jpg?50}}
{{http://www.bluetrait.com/images/N2200/8363c.jpg?50}}
{{http://www.bluetrait.com/images/N2200/8363d.jpg?50}}

### Per Platform Preparations
The following series of SUN Sparcstations require the boot-file name to end with .SUN4M extension.
  * [[howtos:networking:netboot:install:sparc_station:content|Sparcstation 5/10/20]]
  * [[howtos:networking:netboot:install:i386_and_amd64:content]]
  * [[howtos:networking:netboot:client:openbsd:content]]

## References
  * [[http://ezine.daemonnews.org/200604/netvista_s40.html|NetVista]]
