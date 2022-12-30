---
---

# QEMU on OpenBSD

```
qemu-system-i386 -smp 1 -drive file=harddisk.qcow2,if=sd,media=disk,cache=writeback,aio=native -cdrom /mnt/usb/debian-6.0.6-i386-CD-1.iso -m 512 \
-k de -localtime -usb -usbdevice tablet -name debian -display vnc=192.168.1.1:0 -vga std -no-acpi -no-fd-bootchk -balloon none \
-net nic,vlan=0,model=e1000,name=netif_debian -net  tap,vlan=0,name=netif_hosttun1,ifname=tun1,script=no,downscript=no
```