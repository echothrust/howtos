---
---

# Notes for OpenBSD ntpd under VMware ESXi
<sub>This is quite outdated by now. We havent tried any of those things for quite some time...<sub>

The following document outlines information required in order to properly
configure our ESXi hypervisor and OpenBSD guest systems to correctly keep track
of time and time changes through OpenNTPD.

Please note that the following is based on our own experiments/experiences and
outlines what seems to be working best on our environment and setup, your
mileage may vary.

## Introduction
Virtualization allows to run multiple operating systems at the same time on a
single host (hypervisor), however we often forget that this comes at a cost.
The cost in our case is the queuing and scheduling of certain events, and one of
those is the ntp get and set time operations.

On a hypervisor running multiple guests the ntp operations might be queued up,
causing constant increments of the deviation and eventually starving the system
by overloading the hypervisor with sync requests.


## hypervisor configuration
Starting the the hypervisor we need to make sure ESXi is configured in the
following way.
* Runs ntp to sync its local time
* The esxi must be configured to use the `UTC` timezone

ESXi can provide periodical syncing of time of the guests with the hypervisor
through the use of the VMware Tools. In order to check the status of this
periodic sync we have to check the following parameter from the guest `vmx`
configuration file.

For instance for our server named `example.echothrust.com` VMware tools are NOT
configured to periodically sync time:

```sh
grep syncTime /vmfs/volumes/datastore1/example.echothrust.com/*vmx

tools.syncTime = "FALSE"
```

Even if this synchronization is disabled, by default VMware Tools synchronizes
the virtual machine’s time after a few specific events.

This one-time synchronization is done by VMware Tools for specific events such
as:

 * VMware Tools startup (including startup / reboot of the VM)
 * Snapshot operations (creation, resuming)
 * Resuming of suspend
 * vMotion
 * Shrink a virtual disk

### ESXi vSphere Client
Make sure that the system that runs the vSphere client is configured to use UTC
as timezone, otherwise your hypervisor will "change" time for you hypervisor
when it connects and also for your guest systems as well.

Copy paste the following into a file named UTC.reg and import into your system,
in order to change the timezone into UTC.
```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation]
“RealTimeIsUniversal”=dword:00000001
```

## OpenBSD guest configuration
Configuration of an OpenBSD guest is quite easy and straightforward.

Configure `ntpd` to use the existing `vmt` devices as time-delta source by using
the following `/etc/ntpd.conf`.
```
sensor vmt0
```

Activate `ntpd` and we configure it to start with the ability to skip as much
time as it needs in order to sync the clock upon `init`.

```
rcctl enable ntpd
rcctl set ntpd flags "-s"
```

Start the service
```
/etc/rc.d/ntpd start
```

### Notes
* Perform `ntpctl -sa` to check the status of the ntp synchronization.
* OpenBSD syncs the time back to the hardware clock at `shutdown`
* OpenBSD reads the time from the hardware clock at `boot`
* In case of Dell server the iDRAC must also be configured with ntp
* In case of Dell server the iDRAC must also be configured with `UTC`

#### `VMT(4)`

vmt will log notifications that the guest has been suspended or resumed by the
host. It also provides access to the host machine's clock as a timedelta sensor.

A vmwh helper program is available via packages(7).

**NOTE:** The vmwh utility will segfault on a system that isn't run as guest on
the ESXi.


## References
* ntpd(8) http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man8/ntpd.8?query=ntpd
* ntpd.conf(5) http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man5/ntpd.conf.5?query=ntpd.conf&sec=5
* ntpctl(8) http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man8/ntpctl.8?query=ntpctl&sec=8
* rdate(8) http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man8/rdate.8?query=rdate&sec=8
* http://support.ntp.org/bin/view/Support/KnownOsIssues#Section_9.2.2.
* http://support.ntp.org/bin/view/Support/VMWareNTP
* http://www.vmware.com/files/pdf/Timekeeping-In-VirtualMachines.pdf MUST READ
* http://cloudmaniac.net/ntp-esxi/
