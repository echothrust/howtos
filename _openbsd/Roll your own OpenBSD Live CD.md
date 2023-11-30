---
Author: Pantelis Roditis
Based: Based on a guide that I cant find online :(
Created: 2006/08/14 01:45
---

# Roll your own OpenBSD Live CD
- [Introduction](#introduction)
- [Create the main system](#create-the-main-system)
- [Configuring basic system](#configuring-basic-system)
- [Fixing backups](#fixing-backups)
- [Configuring Memory Filesystems](#configuring-memory-filesystems)
- [Create the devices we need to boot](#create-the-devices-we-need-to-boot)
- [Compiling new kernel](#compiling-new-kernel)
- [Patching the Makefile.inc](#patching-the-makefileinc)
- [fstab creation](#fstab-creation)
- [Create an ISO](#create-an-iso)

## Introduction
Since there isn't (unfortunately) an official OpenBSD Live CD we will create one.

## Create the main system
We need a current system and create a release with source code release.
Alternatively you could use OpenBSD stable/release if you have the source code available.

  * Create a directory, this will become root '/' on the CD.
    - `mkdir -p /usr/livecd/backups/dev`
    - `cp baseXX.tgz /usr/livecd/ && cd /usr/livecd/ && tar pxzf baseXX.tgz`
  * Edit/Create/Configure the following files
    - `etc/motd`
    - `etc/mygate`
    - `etc/myname`
    - `etc/sysctl.conf`
    - `etc/rc.conf`
    - `etc/defaultdomain`
    - `etc/hostname.*`
    - `etc/resolv.conf`
    - `etc/hosts`
    - `etc/X11/xorg.conf`
      **WARNING**: The settings should be fairly generic, especially /etc/X11/xorg.conf should use the vesa driver and a maximum resolution of `1024x768`!
  * Fix the keyboard layout by editing the file `etc/kbdtype`
  * Fix the timezone by creating a symbolic link to `etc/localtime`
    - `rm etc/localtime && ln -s usr/share/zoneinfo/Europe/London/localtime`
  * Grab an empty hard drive and make a fresh nice and SLIM install of OpenBSD. As said above you need the source code to the version you install!

**HINT:** Against all good practices ONLY create an **`a`** partition since it will make creating the CD much more easier than having multiple partitions. This includes all packages/ports you want to be on the CD. You should configure the system EXACTLY like you want it to be on CD.

## Configuring basic system
Now mount this partition with another OpenBSD system in order to create a (compressed) tar archive.

**NOTE**: Do not forget the `p` flag!

* Create a compressed image of the installation \
  `cd /mnt/ && tar pczf ~/livecd_root.tar.gz *`
* We transfer this archive to our build machine and extract into our livecd directory we created earlier \
  `tar pxzf livecd_root.tar.gz -C /usr/livecd/`
* We have to copy "/var", "/etc", "/dev", "/root" and "/home" from "/usr/livecd" to "/usr/livecd/backup"
  ```shell
  cp -pR /usr/livecd/{var,etc,root,home} /usr/livecd/backups/
  cp -pR /usr/livecd/dev/MAKEDEV /usr/livecd/backups/dev/
  ```

## Fixing backups
Since a CD is not huge we will compress the _`backup`_ directories into compressed tar archives:
```
cd /usr/livecd/backups
tar pzcf var.tar.gz var
tar pzcf etc.tar.gz etc
tar pzcf dev.tar.gz dev
tar pzcf home.tar.gz home
tar pzcf root.tar.gz root/.[a-z]*
rm -rf /usr/livecd/backups/{var,etc,dev,home,root}
```

## Configuring Memory Filesystems
We have to create virtual partitions in memory (MFS) since we want them to be faster and more importantly writeable. On boot the contents of the archives under `/livecd/backups` is extracted into the memory filesystems we will create.

We have to modify the `etc/rc` script in order for this to work.

**NOTE**: Insert this **after** the line `rm -f /fastboot # XXX (root now writeable)`

```
# Create/mount mfs partitions
echo 'mounting mfs'
mount_mfs -s 51200 -o async,nosuid,nodev,noatime swap /var
mount_mfs -s 6144 -i 4096 -o async,nosuid,nodev,noatime swap /etc
mount_mfs -s 2048 -i 128 -o async,noatime swap /dev
mount_mfs -s 6144 -o async,nosuid,nodev,noatime swap /tmp
mount_mfs -s 8192 -o async,nosuid,nodev,noatime swap /home
mount_mfs -s 8192 -o async,nosuid,nodev,noatime swap /root

# Seems that a short break is necessary here
sleep 2

# Copy over all stuff in mfs partitions
echo -n 'copying files: var '
tar pzxf /backups/var.tar.gz -C /
echo -n 'etc '
tar pzxf /backups/etc.tar.gz -C /
echo -n 'dev '
tar pzxf /backups/dev.tar.gz -C /
echo -n 'home '
tar pzxf /backups/home.tar.gz -C /
echo -n 'root'
echo '.'
tar pzxf /backups/root.tar.gz -C /
echo 'creating device nodes'
cd /dev && sh MAKEDEV all

# Set correct permissions for tmp dirs
test -d /var/tmp && chmod 1777 /var/tmp
test -d /tmp && chmod 1777 /tmp
```

In the same file `etc/rc` add the following after the line `if [ -f /sbin/kbd -a -f /etc/kbdtype ]; then`
```
# We need a root Password
echo 'Need to set a root password'
passwd
# We need a password for our default user as well
echo "Need to set a password for default user 'myuser'"
passwd myuser
```

And at the very end of the file add
```
# Start X environment?
echo 'Do you want to have a [G]raphical environment or [C]onsole only?'
read ans
if [ "$ans" == "G" -o "$ans" == "g" -o "$ans" == "Graphical" ] ; then
    xdm
fi
```

## Create the devices we need to boot
NOTE: not all of them would be necessary, but they don't hurt either since we mount a mfs partition on the real /dev and create devices on boot
```
cd /usr/livecd/dev && ./MAKEDEV all
```

## Compiling new kernel
WARNING: As said several times use the source MATCHING YOUR BINARIES! Bad things will happen if your kernel is not in sync with userland! Now we need to compile a modified GENERIC kernel that is able to boot from CD, we also have to compile it with RAMDISK.

  cd /usr/src/sys/arch/$arch/conf
  mv RAMDISK_CD RAMDISK_CD.old && cp GENERIC RAMDISK_CD

Make the following changes to the RAMDISK_CD you just created.

```
# config bsd swap generic < - we have to comment this one out
option RAMDISK_HOOKS
option MINIROOTSIZE=3800
config bsd root on cd0a
```

Compile the modified kernel.

   config RAMDISK_CD && cd ../compile/RAMDISK_CD/ && make clean && make depend && make

Copy the compiled kernel in the root directory of livecd:

  cp bsd /usr/livecd && chown root:wheel /usr/livecd/bsd && chmod 644 /usr/livecd/kernel

## Patching the Makefile.inc
Apply this patch to the {i386,amd64} `Makefile.inc`.

**NOTE**: You MIGHT have to adjust it since the line numbers may have changed over time. As you see you **HAVE** to adjust it for amd64 as well! `patch-livecd.inc`

```
--- Makefile.inc.orig Thu Nov 25 23:02:08 2004
+++ Makefile.inc Thu Jan 19 13:57:49 2006
-33,8 +33,7
    newfs -m 0 -o space -i 524288 -c 80 ${VND_RDEV}
    mount ${VND_DEV} ${MOUNT_POINT}
    cp ${BOOT} ${.OBJDIR}/boot
- strip ${.OBJDIR}/boot
- strip -R .comment ${.OBJDIR}/boot
+ strip -s -R .comment -K cngetc ${.OBJDIR}/boot
    dd if=${.OBJDIR}/boot of=${MOUNT_POINT}/boot bs=512
    dd if=bsd.gz of=${MOUNT_POINT}/bsd bs=512
    /usr/mdec/installboot -v ${MOUNT_POINT}/boot \
-54,8 +53,7

 bsd.gz: bsd.rd
    cp bsd.rd bsd.strip
- strip bsd.strip
- strip -R .comment bsd.strip
+ strip -s -R .comment -K cngetc bsd.strip
    gzip -c9 bsd.strip > bsd.gz

 bsd.rd: ${IMAGE} bsd rdsetroot
```

Apply by doing:

  cd /usr && patch -p0 < patch-livecd.inc

We need "crunch" to be installed:

  cd /usr/src/distrib/crunch && make obj depend all install

Create the boot image used for the CD. **WARNING**: `/dev/svnd0c` MUSTN'T be in use for this step!

  cd /usr/src/distrib/i386/ramdisk_cd && rm -rf obj/* && make

We have to delete "obj" directory otherwise it will not be updated (BAD!)

Copy the successfully created boot image to livecd directory. **NOTE**: XX is the version like 38

  cp /usr/src/distrib/i386/ramdisk_cd/obj/cdromXX.fs /usr/livecd/

## fstab creation
We have to modify these files in order to be able to boot: `/usr/livecd/etc/fstab`

```
/dev/cd0a / cd9660 ro,noatime 0 0
swap /dev mfs rw,noatime,-s=12000 0 0
swap /tmp mfs rw,noatime,-s=262144 0 0
swap /var mfs rw,noatime,-s=262144 0 0
swap /root mfs rw,noatime,-s=262144 0 0
swap /home mfs rw,noatime,-s=262144 0 0
swap /etc mfs rw,noatime,-s=131072 0 0
```


## Create an ISO
Finally we can create the CD .iso image:
```shell
cd /usr/livecd
mkisofs -vrTJV 'OBSDLive' -b cdromXX.fs -c boot.catalog -R -o /tmp/livecd.iso /usr/livecd
```