# Setting up Network UPS Tools (nut) on OpenBSD

## Installation

Install nut package by issuing `pkg_add -vi nut`.

As explained in /usr/local/share/doc/pkg-readmes/,  the operation of nut is 
split into two daemons:

* `upsd`: Runs on machines directly connected to UPS hardware (e.g. via usb or 
  serial). 
* `upsmon`: Runs on machines for automatic shutdown. Can be configured to 
  operate in SLAVE mode on machines that do not have direct communication with 
  their UPS. Stand-alone setups (single UPS on each server) run `upsmon` in 
  MASTER mode.

For USB connected UPS, make sure `_ups` group has permissions in accessing the
`/dev/ugen*` and `/dev/usb*`:

```sh
# use "sudo usbdevs -vd" if you want exact device
chgrp _ups /dev/ugen* /dev/usb*
```

Create directory for FIFO pipe and lockfile used by timer helper program 
`upssched(8)`:

```
install -d -o _ups -g wheel -m 750 /var/db/nut/upssched
```


## Configuration

This section describes modifications of files in `/etc/nut`.

### ups.conf
This file holds details for UPS devices connected to this host. 

`upsd(8)` uses this file to connect to the hardware. `upsmon(8)` accesses 
devices by their name in this file.

Can be RS232/USB or LAN devices, but also a networked nut server.

Run `nut-scanner` to scan for the UPS:

```
# nut-scanner
Cannot load SNMP library (libnetsnmp) : file not found. SNMP search disabled.
Cannot load XML library (libneon) : file not found. XML search disabled.
Cannot load AVAHI library (libavahi-client) : file not found. AVAHI search disabled.
Scanning USB bus.
No start IP, skipping NUT bus (old connect method)
[nutdev1]
        driver = "blazer_usb"
        port = "auto"
        vendorid = "0665"
        productid = "5161"
        product = "USB to Serial"
        vendor = "Cypress Semiconductor"
        bus = "001"
```

In the case above, the device was detected as "blazer_usb". Append the ini 
formatted output to `/etc/nut/ups.conf`.

### upsd.users
This file contains authentication credentials for `upsd` daemon. 

Add the following users:

```
[admin]
        password = adm1n
        actions = SET
        actions = FSD
        instcmds = ALL

[monmaster]
        password  = m0nmaster
        upsmon master

[monslave]
        password = upsmonslave
        upsmon slave
```

### upsmon.conf

Devices that are monitored by `upsmon` are defined here. This file also 
contains the passwords used to authenticate with `upsd(8)`.

Add the following to `/etc/nut/upsmon.conf` (this is not a full config, just 
local additions):

```
RUN_AS_USER _ups

MONITOR nutdev1@localhost 1 monmaster m0nmaster master

NOTIFYCMD /usr/local/sbin/upssched

NOTIFYFLAG ONLINE     SYSLOG+WALL+EXEC
NOTIFYFLAG ONBATT     SYSLOG+WALL+EXEC
NOTIFYFLAG LOWBATT    SYSLOG+WALL+EXEC
NOTIFYFLAG FSD        SYSLOG+WALL+EXEC
NOTIFYFLAG COMMOK     SYSLOG+WALL+EXEC
NOTIFYFLAG COMMBAD    SYSLOG+WALL+EXEC
NOTIFYFLAG SHUTDOWN   SYSLOG+WALL+EXEC
NOTIFYFLAG REPLBATT   SYSLOG+WALL+EXEC
NOTIFYFLAG NOCOMM     SYSLOG+WALL+EXEC
NOTIFYFLAG NOPARENT   SYSLOG+WALL+EXEC

POLLFREQ 1
POLLFREQALERT 1
```

Device names are derived from ups.conf, in this example the device monitored was named "nutdev1".

upsmon(8) is configured to run upssched via the NOTIFYCMD config directive and the +EXEC NOTIFYFLAGs.

### upssched.conf

This config is used by the upssched(8) helper. It is essentialy a timer helper for scheduling actions based on events from upsmon.

```
CMDSCRIPT /etc/nut/upssched-cmd.sh

PIPEFN /var/db/nut/upssched/upssched.pipe

LOCKFN /var/db/nut/upssched/upssched.lock

# ETS ShutdownIn60Seconds config
# Syntax: AT <notifytype> <upsname> <command>
# notifytypes: ONLINE,ONBATT,LOWBATT,COMMOK,COMMBAD,NOCOMM,FSD,REPLBATT,NOPARENT
# commands: EXECUTE, START-TIMER, CANCEL-TIMER
AT ONBATT * EXECUTE onbatt
AT ONBATT * START-TIMER onbatt-halt 60
AT ONLINE * CANCEL-TIMER onbatt-halt
AT ONLINE * EXECUTE online
AT LOWBATT * EXECUTE lowbatt-halt
AT COMMBAD * EXECUTE commbad
AT COMMOK * EXECUTE commok
AT NOCOMM * EXECUTE nocomm
```

The custom notify script `/etc/nut/upssched-cmd.sh` is included in the next 
section.

## upssched-cmd.sh

Create the script `/etc/nut/upssched-cmd.sh` with the following contents:

```
# This script should be called by upssched via the CMDSCRIPT directive.
# The first argument passed to your CMDSCRIPT is the name of the timer
# from your AT lines in /etc/nut/upssched.conf 
#
# Shutdown delay can be changed from upssched.conf (search for START-TIMER)

# TODO on getting waiting seconds to use in messages from upssched.conf
#WAITSECS=$(grep "START-TIMER onbatt-halt" /etc/nut/upssched.conf | awk '{print $NF}')
WAITSECS=60

LOGCMD="/usr/bin/logger -t upssched-cmd"
LOGFACILITY=local0
# Log Severity can be: panic,alert,crit,error,warn,notice,info,debug

do_shutdown ()
{
        $LOGCMD -p ${LOGFACILITY}.debug "Got to do_shutdown"

        # Connect to other machine and shutdown, e.g:
        #ssh -i /etc/nut/shutdown.key sysadmin@othermachine "sudo halt"

        # Send the shutdown signal to master (will broadcast to all slaves)
        upsmon -c fsd
}

case $1 in
        commbad)
                $LOGCMD -p ${LOGFACILITY}.error "UPS communications failure"
                ;;
        commok)
                $LOGCMD -p ${LOGFACILITY}.notice "UPS communications restored"
                ;;
        nocomm)
                $LOGCMD -p ${LOGFACILITY}.error "UPS communications cannot be established."
                ;;
        onbatt)
                $LOGCMD -p ${LOGFACILITY}.warn "UPS on battery. Shutdown in ${WAITSECS} seconds...."
                ;;
        onbatt-halt)
                $LOGCMD -p ${LOGFACILITY}.error "UPS has been on battery for ${WAITSECS} seconds. Shutting down NOW!!!!"
                do_shutdown
                ;;
        lowbatt-halt)
                $LOGCMD -p ${LOGFACILITY}.crit "UPS battery level CRITICAL. Shutting down NOW!!!!"
                do_shutdown
                ;;
        online)
                $LOGCMD ${LOGFACILITY}.notice "UPS on line. Shutdown aborted."
                ;;
        *)
                $LOGCMD ${LOGFACILITY}.error "Unrecognized command: $1"
                ;;
esac
```

Make sure the script is readable and executable by the nut user `_ups`:

```
chown _ups:wheel /etc/nut/upssched-cmd.sh
chmod 550 /etc/nut/upssched-cmd.sh
```

## Services startup

Make sure the services are started in the correct order:

```
rcctl enable upsd
rcctl enable upsmon
rcctl order upsd
rcctl restart upsd
rcctl restart upsmon
```

## Testing

### Query UPS device

List configured UPS devices:

```
# upsc -l
nutdev1
```

Run `upsc <ups_name>` to dump information from the device, in this case:

```
# upsc nutdev1
battery.charge: 100
battery.voltage: 13.60
battery.voltage.high: 13.00
battery.voltage.low: 10.40
battery.voltage.nominal: 12.0
device.type: ups
driver.name: blazer_usb
driver.parameter.bus: 001
driver.parameter.pollinterval: 2
driver.parameter.port: auto
driver.parameter.product: USB to Serial
driver.parameter.productid: 5161
driver.parameter.vendor: Cypress Semiconductor
driver.parameter.vendorid: 0665
driver.version: 2.7.2
driver.version.internal: 0.11
input.current.nominal: 3.0
input.frequency: 49.9
input.frequency.nominal: 50
input.voltage: 231.8
input.voltage.fault: 231.8
input.voltage.nominal: 230
output.voltage: 231.8
ups.beeper.status: enabled
ups.delay.shutdown: 30
ups.delay.start: 180
ups.load: 0
ups.productid: 5161
ups.status: OL
ups.temperature: 25.0
ups.type: offline / line interactive
ups.vendorid: 0665
```

### UPS device commands

List supported commands on UPS device:

```
# upscmd -l nutdev1    
Instant commands supported on UPS [nutdev1]:

beeper.toggle - Toggle the UPS beeper
load.off - Turn off the load immediately
load.on - Turn on the load immediately
shutdown.return - Turn off the load and return when power is back
shutdown.stayoff - Turn off the load and remain off
shutdown.stop - Stop a shutdown in progress
test.battery.start - Start a battery test
test.battery.start.deep - Start a deep battery test
test.battery.start.quick - Start a quick battery test
test.battery.stop - Stop the battery test
```

You can use `upscmd(8)` to have the UPS execute any commands supported, e.g.:

```
# upscmd -u admin -p adm1n nutdev1 test.battery.start.quick
OK
```

## Troubleshooting

### upsd does not start on reboot, UPS device not detected by kernel.

On the test environment, the UPS recognised as "Cypress Semiconductor USB to Serial" 
was not detected after a reboot. Unplugging & plugging the USB to the same port
did not result in any kernel messages. Plugging to another port would cause the
kernel to detect the device.

In such occasion, creating a crude attach script for hotplugd in 
`/etc/hotplug/attach`, might help:

```
#!/bin/sh

DEVCLASS=$1
DEVNAME=$2

case $DEVCLASS in
    0)
        # generic devices
        case $DEVNAME in
            ugen*)
                # Check that it's actually a "USB to Serial(0x5161)" device
                # made from vendor "Cypress Semiconductor(0x0665)"
                MYVENDOR="0x0665"
                MYDEVICE="0x5161"
                USBVEN="$(usbdevs -dv|grep -B1 ugen0|head -1| cut -d '(' -f3 | cut -d ')' -f1)"
                USBDEV="$(usbdevs -dv|grep -B1 ugen0|head -1| cut -d '(' -f2 | cut -d ')' -f1)"
                if [ "$USBVEN" = "$MYVENDOR" -a "$USBDEV" = "$MYDEVICE" ]; then
                    # Found UPS on ${DEVNAME}, activating upsd(8)
                    logger -t hotplug-attach "Found UPS on ${DEVNAME}, checking upsd."
                    if ! pgrep upsd >/dev/null; then
                        logger -t hotplug-attach "upsd not running. Starting now..."
                        ksh /etc/rc.d/upsd start
                    else
                        logger -t hotplug-attach "Not starting upsd (running already)."
                    fi
                fi
                ;;
        esac
        ;;
esac
```
