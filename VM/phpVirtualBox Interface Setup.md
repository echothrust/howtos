# phpVirtualBox Interface Setup

## Description
This web application is a PHP front for Oracle VirtualBox running headless. It connects to `vboxwebsrv`, a SOAP server distributed with VirtualBox, and provides a web interface that mimics VirtualBox.

## Known security risks
phpVirtualBox uses system account 'vbox' to communicate SOAP in PLAINTEXT with vboxwebsrv on 127.0.0.1:18083. That is why we run them on the same box.

## Server access
* 172.16.10.10 (r00t.echothrust.dev)
* https://r00t.echothrust.dev/vboxweb uid/pw: `admin`/`admin`

## Supporting services

### VirtualBox
This is the VM Host application (not a real service), but it is the base application package for `vboxwebsrv`. Some extra configuration on VirtualBox is needed:

* Oracle VM VirtualBox Extension Pack
** Available through: https://www.virtualbox.org/wiki/Downloads
** Installation command (as root): `VBoxManage extpack install /path/to/extpack`

* For console access via phpVirtualBox
** command `VBoxManage list extpacks` as user 'vbox' should return
>vbox@r00t:~$ VBoxManage list extpacks
Extension Packs: 1
Pack no. 0:   Oracle VM VirtualBox Extension Pack
[...]
VRDE Module:  VBoxVRDP
Usable:       true

** In case console is unavailable for a specific guest VM, use commands (as user 'vbox'):
>`VBoxManage list vms` to get the name,
and `VBoxManage modifyvm "VM name" --vrde on` to enable VRDP on the specific guest

### vboxwebsrv
VirtualBox's SOAP web service

* Init script
> /etc/init.d/vboxweb-service

* Configuration file /etc/default/virtualbox
> VBOXWEB_USER=vbox
VBOXWEB_LOGFILE=/var/log/vboxweb/vboxwebsrv.log
#Usefull if setting up phpVirtualBox + vboxwebsrv on different hosts
#VBOXWEB_HOST=127.0.0.1 # IP to bind to
#VBOXWEB_PORT=18083 # Port to bind to

###  vbox-autostart
This "service" runs once on-boot to start any virtual machines that are set to autostart.

* Init script
> /etc/init.d/vboxautostart-service

* Configuration file /etc/default/virtualbox
>VBOXAUTOSTART_DB=/etc/vbox
VBOXAUTOSTART_CONFIG=/etc/vbox/vbox-autostart.cfg

* Directory permissions
> chgrp vboxusers /path/to/$VBOXAUTOSTART_DB
chmod 1770 /path/to/$VBOXAUTOSTART_DB

* Configuration file /etc/vbox/vbox-autostart.cfg
>#Default policy is to deny starting a VM, the other option is "allow".
default_policy = deny
#Create an entry for each user allowed to run autostart
vbox = {
allow = true
}

* Shell script for setting autostart & autostart delays on all VMs (run as user 'vbox')
> INITIAL_DELAY=30
DELAY_INTERVAL=60
echo "Warning: First power off all VMs. After that, copy & paste the following commands:"
for i in $(VBoxManage list vms|cut -d' ' -f1)
do
	echo VBoxManage modifyvm ${i} --autostart-enabled on ;
	echo VBoxManage modifyvm ${i} --autostart-delay ${INITIAL_DELAY};
	INITIAL_DELAY=$((INITIAL_DELAY+DELAY_INTERVAL))
done

* Checking Autostart settings for a specific VM:
    vbox@r00t:~$ VBoxManage showvminfo puppet |grep -i auto
    Autostart Enabled: on
    Autostart Delay: 150

###  VMs gracefull auto-shutdown
Acpi signal is sent by stock init script (and waits 30 seconds for guests to shutdown). Configuration file /etc/default/virtualbox.
>SHUTDOWN_USERS="vbox"
SHUTDOWN=acpibutton

### On-demand poweron & acpi-shutdown for local VMs
Added the following file into the web root.
$ `cat /var/www/vboxweb/control.php`

```php
<?php
/*
 /etc/sudoers: www-data  ALL= (vbox) NOPASSWD : /usr/bin/VBoxManage
 sudo -u vbox -H command_to_run
 VBoxManage showvminfo "devdb"|grep "^State: *.*"
 VBoxManage controlvm "devdb" acpipowerbutton
 VBoxManage startvm "devdb" --type headless
*/

if (!isset($_REQUEST['host']) || !isset($_REQUEST['action']) ) {
  echo "Bad 'host'{vm_guest_name} and 'action'{on,off} in _REQUEST (POST/GET)";
}
else {
  $host = escapeshellcmd($_REQUEST['host']);
  $check = exec ('sudo -u vbox -H VBoxManage showvminfo "'.$host.'" |grep "^State: *.*"');
  switch($_REQUEST['action']) {
    case "on":
      if(preg_match('/^State: *powered off.*/',$check)) {
        if ( isset($_REQUEST['snapshot']) ) {
          $snapshot=escapeshellcmd($_REQUEST['snapshot']);
          exec ('sudo -u vbox -H VBoxManage snapshot "'.$host.'" restore "'.$snapshot.'"');
        }
        exec ('sudo -u vbox -H VBoxManage startvm "'.$host.'" --type headless');
      }
      else echo "Could not complete: host not in powered off state.";
      break;
    case "off":
      if(preg_match('/^State: *running.*/',$check))
        exec ('sudo -u vbox -H VBoxManage controlvm "'.$host.'" acpipowerbutton');
      else echo "Could not complete: host not in running state.";
      break;
    default:
      echo "action can be {on,off}";
      break;
  }
}

```
