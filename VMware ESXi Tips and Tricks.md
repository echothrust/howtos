# TITLE:VMware ESXi Tips and Tricks
**Author:** Pantelis Roditis\
**Created:** 2010/12/07

## Change hostname
* edit /etc/hosts
* Run the commands
```
esxcli system hostname set --host=hostname
esxcli system hostname set --fqdn=fqdn
```

## Create Raw Disk mappings
Find the disk conroler under /vmfs/devices/disks/vmhbaXX:0:0:0:0 and create a raw disk mapping inside the folder you want.
```
vmkfstools -a lsilogic -r /vmfs/devices/disks/vmhba35:0:0:0 DISK1TB-NEW.vmkd
```


## Create a new VM from the command line
* Create the folder that will hold the virtual machine
```
mkdir /vmfs/volumes/datastore1/demoserver
cd /vmfs/volumes/datastore1/demoserver
```

* Create the virtual disk
```
vmkfstools -c 8G -a lsilogic demoserver.vmdk
```

* Create the vmx configuration file for the vm
```
cat << __EOF__ > demoserver.vmx
config.version = "8"
virtualHW.version = "4"
vmci0.present = "TRUE"
displayName = "demoserver"
floppy0.present = "FALSE"
numvcpus = "1"
scsi0.present = "TRUE"
scsi0.sharedBus = "none"
scsi0.virtualDev = "lsilogic"
memsize = "256"
scsi0:0.present = "TRUE"
scsi0:0.fileName = "demoserver.vmdk"
scsi0:0.deviceType = "scsi-hardDisk"
ethernet0.present = "TRUE"
ethernet0.virtualDev = "vmxnet"
ethernet0.features = "15"
ethernet0.networkName = "VM Network"
ethernet0.addressType = "generated"
guestOS = "other-64"
RemoteDisplay.vnc.enabled = "True"
RemoteDisplay.vnc.port = "5901"
RemoteDisplay.vnc.password = "secure"
guestOSAltName = "OpenBSD 4.8 amd64"
__EOF__
```

* Change the permissions of the vmx file so that it will work
```
chmod a+rwx demoserver.vmx
```


**RemoteDisplay.vnc.port** Must be one that is not currently been used.

## List current VMs and their ID
```
vim-cmd vmsvc/getallvms
```
## Register/Unregister
```
vim-cmd solo/registervm /vmfs/volumes/datastore1/demoserver/demoserver.vmx
vim-cmd vmsvc/destroy 10
```
## Power On/Off
```
vim-cmd vmsvc/power.on 10
vim-cmd vmsvc/power.off 10
```

## Create a disk to be clustered
```sh
vmkfstools -c 12G -a lsilogic -d eagerzeroedthick DATA1.vmdk
vmkfstools -c 12G -a lsilogic -d eagerzeroedthick DATA2.vmdk
vmkfstools -c 12G -a lsilogic -d eagerzeroedthick RECO1.vmdk
vmkfstools -c 12G -a lsilogic -d eagerzeroedthick RECO2.vmdk
vmkfstools -c 5G -a lsilogic -d eagerzeroedthick raw5.vmdk
vmkfstools -c 5G -a lsilogic -d eagerzeroedthick raw6.vmdk
```

## Syslog configuration

For more information regarding the use of esxcli, see Configuring ESXi Syslog Services in the vSphere Command-Line Interface Documentation.

Open a ESXi Shell console session where the esxcli command is available, such as the vCLI or on the ESXi host directly.
Display the existing five configuration options on the host using the command:

```
esxcli system syslog config get
```

Set new host configuration, specifying options to change, using a command similar to:

```
esxcli system syslog config set --logdir=/path/to/vmfs/directory/ --loghost=RemoteHostname --logdir-unique=true|false --default-rotate=NNN --default-size=NNN
```

For example, to configure remote syslog using TCP on port 514:

```
esxcli system syslog config set --loghost='tcp://10.11.12.13:514'
```

To configure remote syslog using UDP on port 514:

```
esxcli system syslog config set --loghost='udp://10.11.12.13:514'
```

Note: In ESXi 5.0, you must download a patch on the ESXi host if you are using syslog with UDP. For more information, see VMware ESXi 5.0, Patch ESXi-5.0.0-20120704001-standard (2019113).


After making configuration changes, load the new configuration using the command:

```
esxcli system syslog reload
```

Note: This command may be used to restart the syslog service if and when the service is stopped.


## Enable VNC Access to ESXi VMs
We can enable VNC connectivity to our virtual machines in ESXi.

To enable VNC support, power off your VM and edit your VMX file (you can download it to your PC with Datastore Browser if you don’t want to edit it in place). It is a simple text file. Put these two lines in the file:
```
remotedisplay.vnc.port="5900"
remotedisplay.vnc.enabled="true"
remotedisplay.vnc.password = "yourpassword"
```
Replace `yourpassword` with a password of your choosing, of course.

If you have multiple VMs on a single host, you’ll need to use a different VNC port for each. If you do that, then when you connect with the VNC client, just specify the port after the IP address, just like you would with a special port in a web browser or anything else.


## Patch Procedure for Dell-Customised ISOs

We avoid the usual full update procedure and by using the update manager, we can keep any dell-customized drivers, which would otherwise be lost:

1. As usual, migrate or power off the virtual machines running on the host and put the host into maintenance mode.
```
vim-cmd hostsvc/maintenance_mode_enter
```

2. Allow outgoing connections to ports 80,443
```sh
esxcli network firewall ruleset set -e true -r httpClient
```
3. List available update profiles from VMware's online VUM, for 'ESXi-5.5.0'.
```sh
esxcli software sources profile list -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml |grep 'ESXi-5.5.0'|sort
```
4. After running the command, you will see a listing of available software updates. After finding the system profile from VMware you want to update to, simply run the following command:
```sh
esxcli software profile update -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml -p ESXi-5.5.0-2014mmddnnn-standard
```
5. After the patch has been installed, reset firewall rules from step 2 and reboot the ESX host:
```sh
esxcli network firewall ruleset set -e false -r httpClient && reboot
```
6. After the host has finished booting, exit maintenance mode and power on the virtual machines:
```sh
vim-cmd hostsvc/maintenance_mode_exit
```
