---
---

# TITLE:OpenBSD Netboot Diskless OpenBSD install
**Author:** Pantelis Roditis
**Created:** 2011/01/28

# Introduction
The following paper describes how to setup an OpenBSD system to be able to be booted from a system without a hard drive (Diskless).

* Extract the base system
Decide where your OpenBSD system will be stored and extract your OpenBSD distribution.
```sh
mkdir -p /n/sys/obsd/b
cd /n/sys/obsd/b
for pkg in /n/pub/OpenBSD/4.8/i386/*.tgz;do tar zxvpf $pkg;done
```

* Configure the system
All the following commands assume that the root directory of your new OBSD is under `/n/sys/obsd/b`

* Populate the /dev directory
```sh
cd /n/sys/obsd/b/dev && ./MAKEDEV all
```

* Configure the networking
```sh
cd /n/sys/obsd/b/etc
echo 'inet 172.16.0.1 255.255.255.0 172.16.0.255 NONE'>hostname.if
for ifname in re0 rl0 ne0 bge0 fxp0 age0 nfe0 xl0;do ln -s hostname.if hostname.$ifname;done
echo -e "domain rnd.echothrust.dev\nnameserver 172.16.200.254\norder file bind">resolv.conf
echo "172.16.200.8 tboot">>hosts
echo "172.16.200.254">mygate
echo "obsd">myname
```

* Create your fstab
```sh
cd /n/sys/obsd/b/etc
echo 'tboot:/n/sys/obsd/b / nfs rw 0 0'>fstab
```

* Change the timezone of the system
```sh
ln -sf /usr/share/zoneinfo/Europe/Athens localtime
```

If the extracted system is the same architecture and version as your own you can `chroot` and work based on root (`/etc/resolv.conf`,`/etc/fstab` etc)
