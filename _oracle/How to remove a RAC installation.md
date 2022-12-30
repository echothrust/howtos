---
---

# How to remove a RAC installation

<code>
#rm -rf /etc/oracle
#umount -f /u01
#cat /etc/fstab
#fdisk -l
#mkfs.ext3 -L /u01 /dev/sdb1
#mount -a
#ls -la /u01
#ls -la /etc
#rm -rf /tmp/.oracle /usr/tmp/.oracle
#vi /etc/inittab
#rm -rf /etc/init.d/init.*
#ps -aef|grep ora
#rm /etc/oraInst.loc
#cat /etc/sysconfig/rawdevices
#dd if=/dev/zero of=/dev/sdd1
#dd if=/dev/zero of=/dev/sde1
#reboot
</code>

<code>
mkdir -p /u01/crs/oracle/product/10.2.0/crs
mkdir -p /u01/app/oracle/product/10.2.0/db_1
mkdir -p /u01/oradata
chown -R oracle.oinstall /u01
</code>