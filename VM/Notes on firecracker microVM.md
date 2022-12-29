# Notes on firecracker microVM
Firecracker is a kvm interaction interface similar to qemu. In contrast with
qemu, however, firecracker is only able to boot linux virtual machines and with
only a subset of the supported hardware devices. This in return provides
extremely fast startup times. You can literally have a linux up in a matter of
seconds.

Firecracker works on bare-metal systems and although with some tweaks you can
make it run on a VM itself, it is not recomended. Out produced lots of kernel
panics of the host system.


## Create kernel initrds
Download kernels and build required versions you like. Use the kernel config
provided by firecracker and build the kernels you need before hand.

Firecracker uses initrds to start the linux system up and then switch to the
guest filesystem for the remaining of the boot process (eg init, systemd etc).

## Create filesystems
The easiest way to maintain a firecracker microVM is with the use of existing
Docker images to produce the vm filesystems. Then we can download and use an
existing kernel from official repos (eg debian) or download one and build it.

Make sure the following packages are installed for the container to be able to
turn into a microVM

```sh
sysvinit-core
```

Create target as standard docker

```
IMG_ID=$(docker build -q .)
CONTAINER_ID=$(docker run -td $IMG_ID /bin/bash)

MOUNTDIR=mnt
FS=mycontainer.ext4

mkdir $MOUNTDIR
qemu-img create -f raw $FS 800M
mkfs.ext4 $FS
mount $FS $MOUNTDIR
docker cp $CONTAINER_ID:/ $MOUNTDIR
umount $MOUNTDIR
```

## VM Config template
vm_config.json
```json
{
  "boot-source": {
    "kernel_image_path": "/full/path/to/hello-vmlinux.bin",
    "boot_args": "console=ttyS0 reboot=k panic=1 pci=off"
  },
  "drives": [
    {
      "drive_id": "rootfs",
      "path_on_host": "hello-rootfs.ext4",
      "is_root_device": true,
      "is_read_only": false
    }
  ],
  "machine-config": {
    "vcpu_count": 1,
    "mem_size_mib": 512,
    "ht_enabled": false
  },
  "network-interfaces": [
   {
     "iface_id": "eth0",
     "host_dev_name": "veth0"
   }
  ]
}
```

## Networking
Create a tap device.

```sh
ip tuntap add veth0 mode tap
ip link set tap0 up
```

Then, add it to an existing bridge, something which can route to the internet, e.g. docker0.
```sh
brctl addif docker0 veth0
ip link set dev veth0 up
```
Once the VM is booted, configure networking, or rely on DHCP (if available on the bridge).

```sh
ip addr add dev eth0 172.17.100.1/16
ip route add default via 172.17.0.1
ping 1.1.1.1 -c 3
```
