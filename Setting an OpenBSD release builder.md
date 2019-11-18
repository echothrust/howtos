# Setting up an OpenBSD release builder
A release builder for OpenBSD is quite useful when you have a farm of systems
that you have to maintain.

However, due to the nature of the build system one can only build and maintain
stable updated releases for a particular tree only. This makes the need for two
build systems obligatory. One system will follow the "STABLE" and one build
system will have to follow the "snapshot" in order to avoid kernel
incompatibilities.

Our own stable release builder, is a VirtualBox virtual machine named
`builder58` with IP `172.20.1.1` (this is not a static IP). This
system is powered off most of the time and only boots up in order to fetch an
updated tree and build an up2date release.

XXXFIXMEXXX We need a schedule to perform these tasks (eg daily sync and
rebuild?)  

## Specifications for `release-builder`

Some of the system details to ease in deciding

* One single disk 50GB auto partitioned
```sh
# df -h
Filesystem     Size    Used   Avail Capacity  Mounted on
/dev/wd0a     48.2G    897M   44.9G     2%    /
```

## Fetch the packages

```sh
mkdir -p /home/tgzs && cd /home/tgzs
for i in src sys ports xenocara
do
 ftp ftp://ftp.openbsd.org/pub/OpenBSD/$(uname -r)/$i.tar.gz
done

cd /usr/src
tar zxpf /home/tgzs/src.tar.gz
tar zxpf /home/tgzs/sys.tar.gz
cd /usr
tar zxpf /home/tgzs/xenocara.tar.gz
tar zxpf /home/tgzs/ports.tar.gz
```

* Update from CVS

```sh
export CVSROOT=anoncvs@ftp5.eu.openbsd.org:/cvs
cd /usr
cvs -qd$CVSROOT checkout -rOPENBSD_5_7 -P src ports xenocara
```

Later you can update the repository by executing
```
cd /usr/src && cvs -qd$CVSROOT up -rOPENBSD_5_7 -Pd
cd /usr/xenocara && cvs -qd$CVSROOT up -rOPENBSD_5_7 -Pd
cd /usr/ports && cvs -qd$CVSROOT up -rOPENBSD_5_7 -Pd
```


## Building the new release


```sh
#!/bin/ksh
#
# Perform all operations required to produce a new updated release
# The new release is left under a folder /altroot/YYYYMMDD
export LMAJOR=5
export LMINOR=7
export LBRANCH=OPENBSD_${LMAJOR}_${LMINOR}
export CVSROOT=anoncvs@ftp5.eu.openbsd.org:/cvs

( cd /usr/src && make clean ) || exit 1
(cd /usr/xenocara && make clean) || exit 2
rm -rf /usr/obj/* /usr/dest /usr/xdest /usr/rel

echo "Updating /usr/src"
(cd /usr/src && cvs -qd$CVSROOT up -r${LBRANCH} -Pd) || exit 3

echo "Updating /usr/xenocara"
(cd /usr/xenocara && cvs -qd$CVSROOT up -r${LBRANCH} -Pd) || exit 4
echo "Updating /usr/ports"
(cd /usr/ports && cvs -qd$CVSROOT up -r${LBRANCH} -Pd) || exit 5

cd /usr/src/sys/arch/`machine`/conf
config GENERIC.MP
cd ../compile/GENERIC.MP
make clean && make && make install
cp /sbin/reboot /sbin/oreboot
rm -rf /usr/obj/*
cd /usr/src
make obj
cd /usr/src/etc
env DESTDIR=/ make distrib-dirs
cd /usr/src
make build
export DESTDIR=/usr/dest
export RELEASEDIR=/usr/rel
rm -rf ${DESTDIR} ${RELEASEDIR}
mkdir -p ${DESTDIR} ${RELEASEDIR}
cd /usr/src/etc && make release
cd /usr/src/distrib/sets
sh checkflist
rm -rf /usr/xobj/*
cd /usr/xenocara
make bootstrap
make obj
make build
export DESTDIR=/usr/xdest
export RELEASEDIR=/usr/rel
rm -rf $DESTDIR
mkdir -p $DESTDIR ${RELEASEDIR}
make release
cd $RELEASEDIR
cksum -a sha256 *tgz > SHA256
/bin/ls -nT > index.txt
mv $RELEASEDIR /altroot/$(date +"%Y%m%d")
echo "If /sbin/reboot fails use the /sbin/oreboot command"
```

Once this is finished make sure you copy the newly created release folder
on zeus under the following locations:
 * `/mirror/OpenBSD/x.y/amd64` and
   `cp amd64.orig/installXY.iso amd64/`
 * `/var/www/htdocs/pub/OpenBSD/X.Y/amd64` and
   `cp amd64.orig/installXY.iso /var/www/htdocs/pub/OpenBSD/X.Y/amd64`
