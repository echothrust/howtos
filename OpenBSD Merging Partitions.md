---
author: Pantelis Roditis (databus)
tags:
  - OpenBSD
  - disklabel
---

# Doing it 1999 style: OpenBSD Merging Partitions
This document attempts to outline the steps required to merge multiple partitions back into one.

This is particularly targeting new users to OpenBSD with a few Linuxism's here and there to make relatable.

Most of the information in here is already covered on the OpenBSD FAQ. This is not a replacement for it, this document is simply here to walk you though the process with examples so that you can do the same.

## The setup
On a VM I configured with 16GB, to run some simple tests, the auto layout created the following partitions
```shell
foo# df -h
Filesystem     Size    Used   Avail Capacity  Mounted on
/dev/sd0a      404M   84.1M    299M    22%    /
/dev/sd0k      2.5G    2.0K    2.4G     0%    /home
/dev/sd0d      531M   14.0K    504M     0%    /tmp
/dev/sd0f      1.9G    1.2G    666M    64%    /usr
/dev/sd0g      525M    298M    200M    60%    /usr/X11R6
/dev/sd0h      1.7G    220K    1.6G     0%    /usr/local
/dev/sd0j      5.0G    2.0K    4.8G     0%    /usr/obj
/dev/sd0i      1.5G    2.0K    1.4G     0%    /usr/src
/dev/sd0e      758M    7.5M    712M     1%    /var
foo#
```

Most of the partitions will be fine as is however `/usr/src`, `/usr/obj`, will most likely never be used. However, 1.6GB for `/usr/local` will not be enough to install some eye candy desktop.

## The plan (baby steps)
Our plan is to start with something simple at first and with less chances of losing data. We are going to merge `/usr/src` and `/usr/obj` into the `/usr/local` partition.

First we ensure that the partitions are empty as to avoid messing up something and copy whatever is needed back after the merge.
```shell
foo# ls -la /usr/src
total 8
drwxrwxr-x   2 root  wsrc   512 May 24 03:25 .
drwxr-xr-x  16 root  wheel  512 May 24 00:28 ..
foo# ls -la /usr/obj
total 8
drwxrwx---   2 build  wobj   512 May 24 03:25 .
drwxr-xr-x  16 root   wheel  512 May 24 00:28 ..
foo#
```

Both are empty so we proceed into unmounting them
```shell
foo# unmount /usr/src
foo# unmount /usr/obj
```

Temporarily, we unmount the partition for the `/usr/local` so that we can manipulate it without corruption.
```shell
foo# umount /usr/local
```

If you get an error similar to the following, ensure that you kill any processes that my be using this partition
```shell
foo# umount /usr/local
umount: /usr/local: Device busy
```

Comment out the entries on `/etc/fstab` to avoid being mounted by accident again and to avoid getting error during our next reboot (since we are going to delete those two).
```shell
#9a04e665d7da593d.j /usr/obj ffs rw,nodev,nosuid 1 2
#9a04e665d7da593d.i /usr/src ffs rw,nodev,nosuid 1 2
```

## Let the fun begin
Run the `disklabel` utility and note the offset and sizes of the partitions
```shell
foo# disklabel -E sd0
Label editor (enter '?' for help at any prompt)
```

Display the partition table using human sizes **`p m`** and size in blocks with just **`p`**
```
sd0> p m
OpenBSD area: 64-33554432; size: 16384.0M; free: 0.0M
#                size           offset  fstype [fsize bsize   cpg]
  a:           420.1M               64  4.2BSD   2048 16384  6667 # /
  b:           620.2M           860416    swap                    # none
  c:         16384.0M                0  unused
  d:           552.1M          2130592  4.2BSD   2048 16384  8764 # /tmp
  e:           782.2M          3261376  4.2BSD   2048 16384 12516 # /var
  f:          2040.2M          4863424  4.2BSD   2048 16384 12960 # /usr
  g:           546.0M          9041728  4.2BSD   2048 16384  8667 # /usr/X11R6
  h:          1834.3M         10160032  4.2BSD   2048 16384 12960 # /usr/local
  i:          1608.0M         13916640  4.2BSD   2048 16384 12960
  j:          5336.1M         17209888  4.2BSD   2048 16384 12960
  k:          2644.7M         28138176  4.2BSD   2048 16384 12960 # /home
sd0> p
OpenBSD area: 64-33554432; size: 33554368; free: 16
#                size           offset  fstype [fsize bsize   cpg]
  a:           860352               64  4.2BSD   2048 16384  6667 # /
  b:          1270160           860416    swap                    # none
  c:         33554432                0  unused
  d:          1130784          2130592  4.2BSD   2048 16384  8764 # /tmp
  e:          1602048          3261376  4.2BSD   2048 16384 12516 # /var
  f:          4178304          4863424  4.2BSD   2048 16384 12960 # /usr
  g:          1118304          9041728  4.2BSD   2048 16384  8667 # /usr/X11R6
  h:          3756608         10160032  4.2BSD   2048 16384 12960 # /usr/local
  i:          3293248         13916640  4.2BSD   2048 16384 12960
  j:         10928288         17209888  4.2BSD   2048 16384 12960
  k:          5416256         28138176  4.2BSD   2048 16384 12960 # /home
```

Our partitions that we are interested are **`i`** & **`j`** and we are going to merge them into **`h`**.
* **i** offset: 13916640, size: 3293248
* **j** offset: 10928288, size: 17209888

**Note** that your values will differ, adapt them accordingly

Delete the two partitions and print the table again to confirm.

```shell
sd0> d i
sd0*> d j
sd0*> p
OpenBSD area: 64-33554432; size: 33554368; free: 14221552
#                size           offset  fstype [fsize bsize   cpg]
  a:           860352               64  4.2BSD   2048 16384  6667 # /
  b:          1270160           860416    swap                    # none
  c:         33554432                0  unused
  d:          1130784          2130592  4.2BSD   2048 16384  8764 # /tmp
  e:          1602048          3261376  4.2BSD   2048 16384 12516 # /var
  f:          4178304          4863424  4.2BSD   2048 16384 12960 # /usr
  g:          1118304          9041728  4.2BSD   2048 16384  8667 # /usr/X11R6
  h:          3756608         10160032  4.2BSD   2048 16384 12960 # /usr/local
  k:          5416256         28138176  4.2BSD   2048 16384 12960 # /home
```

Lets change our `h` partition for `/usr/local` to increase its size. Open a calculator and add up the size values for h,i,j
```shell
foo# echo $((3756608+3293248+17209888))
24259744
```

and add the sum to the size question
```shell
sd0*> c h
Partition h is currently 3756608 sectors in size, and can have a maximum
size of 17978144 sectors.
size: [3756608] 24259744
```

Confirm the new sizes
```shell
sd0*> p
OpenBSD area: 64-33554432; size: 33554368; free: 16
#                size           offset  fstype [fsize bsize   cpg]
  a:           860352               64  4.2BSD   2048 16384  6667 # /
  b:          1270160           860416    swap                    # none
  c:         33554432                0  unused
  d:          1130784          2130592  4.2BSD   2048 16384  8764 # /tmp
  e:          1602048          3261376  4.2BSD   2048 16384 12516 # /var
  f:          4178304          4863424  4.2BSD   2048 16384 12960 # /usr
  g:          1118304          9041728  4.2BSD   2048 16384  8667 # /usr/X11R6
  h:         17978144         10160032  4.2BSD   2048 16384 12960 # /usr/local
  k:          5416256         28138176  4.2BSD   2048 16384 12960 # /home
sd0*> p m
OpenBSD area: 64-33554432; size: 16384.0M; free: 0.0M
#                size           offset  fstype [fsize bsize   cpg]
  a:           420.1M               64  4.2BSD   2048 16384  6667 # /
  b:           620.2M           860416    swap                    # none
  c:         16384.0M                0  unused
  d:           552.1M          2130592  4.2BSD   2048 16384  8764 # /tmp
  e:           782.2M          3261376  4.2BSD   2048 16384 12516 # /var
  f:          2040.2M          4863424  4.2BSD   2048 16384 12960 # /usr
  g:           546.0M          9041728  4.2BSD   2048 16384  8667 # /usr/X11R6
  h:          8778.4M         10160032  4.2BSD   2048 16384 12960 # /usr/local
  k:          2644.7M         28138176  4.2BSD   2048 16384 12960 # /home
```

Great our partition is now ~8GB, lets save and quit the utility (note that `q` also saves but force of habit dictates saving first before quitting)
```shell
sd0*> w
sd0> q
No label changes.
```

Great from here on you can follow the remaining steps from the [OpenBSD FAQ - Grow Partition](https://www.openbsd.org/faq/faq14.html#GrowPartition) finish up.

Notice that the FAQ happened to mention the same partition as ours, something tells me this is not by accident

Now lets grow our filesystem to match the new partition definition.
```shell
foo# growfs sd0h
We strongly recommend you to make a backup before growing the Filesystem

 Did you backup your data (Yes/No) ? Yes
new filesystem size is: 4494536 frags
Warning: 145184 sector(s) cannot be allocated.
growfs: 8707.5MB (17832960 sectors) block size 16384, fragment size 2048
        using 43 cylinder groups of 202.50MB, 12960 blks, 25920 inodes.
super-block backups (for fsck -b #) at:
 4147360, 4562080, 4976800, 5391520, 5806240, 6220960, 6635680, 7050400, 7465120, 7879840, 8294560, 8709280, 9124000, 9538720, 9953440, 10368160, 10782880, 11197600, 11612320, 12027040, 12441760, 12856480, 13271200, 13685920, 14100640, 14515360,
 14930080, 15344800, 15759520, 16174240, 16588960, 17003680, 17418400
```

Check the filesystem
```shell
fsck /dev/sd0h
foo# fsck /dev/sd0h
** /dev/rsd0h
** Last Mounted on /usr/local
** Phase 1 - Check Blocks and Sizes
** Phase 2 - Check Pathnames
** Phase 3 - Check Connectivity
** Phase 4 - Check Reference Counts
** Phase 5 - Check Cyl groups
111 files, 110 used, 4318081 free (57 frags, 539753 blocks, 0.0% fragmentation)

MARK FILE SYSTEM CLEAN? [Fyn?] y


***** FILE SYSTEM WAS MODIFIED *****
```

Remount your filesystem and check the size
```shell
foo# mount /usr/local
foo# df -h
Filesystem     Size    Used   Avail Capacity  Mounted on
/dev/sd0a      404M   84.1M    299M    22%    /
/dev/sd0k      2.5G    2.0K    2.4G     0%    /home
/dev/sd0d      531M   14.0K    504M     0%    /tmp
/dev/sd0f      1.9G    1.2G    666M    64%    /usr
/dev/sd0g      525M    298M    200M    60%    /usr/X11R6
/dev/sd0e      758M    7.6M    712M     1%    /var
/dev/sd0h      8.2G    220K    7.8G     0%    /usr/local
```

Congrats you did it... if you liked this guide and want to thank me, please consider supporting the OpenBSD project by making donations or purchases.