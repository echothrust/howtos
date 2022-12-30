---
author: Pantelis Roditis
twitter: "@PantelisRoditis"
tags:
  - OpenBSD
  - pflogd
  - newsyslog
---

# OpenBSD running multiple pflogd instances
OpenBSD provides a nice way to log for firewall packets in a more lightweight, manageable and safe way than just running tcpdump. With `pflogd` you get a nice lightweight daemon to capture PF packets on special interfaces.

These notes ouline the procedure of adding multiple rc control scripts to manage multiple pflogd processes, as well as the necessary changes to `newsyslog.conf` to keep your capture files nice and tidy.

## The issue
Although the procedure of setting up a new pflog interface is fairly simple you need to pay careful attention as certain default settings may cause you confusion during operations.

OpenBSD, by default, provides an rc script (`/etc/rc.d/pflogd`) to handle the daemon. This script uses a regular expression that matches for running `pflogd` instances and requests for them to be restarted or stopped.

Similarly, for rotating logs, OpenBSD uses `/etc/newsyslog.conf` with the following command `pkill -HUP -u root -U root -t - -x pflogd` to request the daemon to close and reopen its log files,
```
/var/log/pflog   600  3     250  *     ZB "pkill -HUP -u root -U root -t - -x pflogd"
```

This command again matches running pflogd instances.

Both of those commands match running pflogd instances, all of them. Even the ones that didnt start from by that rc script or write to that log file.

This led to some situations that we didnt expect.

### The _"Oh shit where's my pflogd"_ situation
Whenever you try to restart one instance all of them will be restarted. This may not be a problem unless you are running multiple pflogd instances that you dont want to restart, reload etc.

Check the next example; here `rcctl` kills the daemon that I had started manually on another terminal.
```shell
foo# rcctl restart pflogd
pflogd(ok) # <-- OUPS
/etc/rc.d/pflogd: need -f to force start since pflogd_flags=NO
foo# grep pflogd /etc/rc.conf.local
pflogd1_flags=-s /var/log/pflog1 -i pflog1 -s 1500 -f /var/log/pflog1
pflogd_flags=NO
```

This is an extremely rare case but... it happened to us at a very unfortunate time.

### The _"Oh shit why my pflogd closed and reopened its logs"_ situation
Maybe less dangerous than the previous case is that of the rotation of logs. Its less dangerous because it only sends a HUP signal.

Worse case scenario with this, you'll only see a couple of packets missing from your capture. Very unlikely to cause problems, but again it may get you off-guard.

## OMG! WHAT DO WE DO!!! OMG!
Well luckily things are fairly easy to modify to our needs and avoid these dreaded situations.

1. Copy the existing `/etc/rc.d/pflogd` in case you mess up (yeah just in case 🤣)
```shell
foo# cp /etc/rc.d/pflogd /root
```
2. Stop any running pflogd processes
```shell
foo# pkill -9 pflogd
```


### First things first

1. Edit the file `/etc/rc.d/pflogd` and modify the `pexp` pattern accordingly (_NOTE: changing the default system files is not a generally good idea..._)
```shell
pexp="pflogd: \[running\]${daemon_flags:+ ${daemon_flags}}"
```
2. Set flags for the default pflogd daemon. These are needed so we can distinguish the processes between them. The flags we are setting are the default used by pflogd.
```shell
foo# rcctl set pflogd flags -s 160 -i pflog0 -f /var/log/pflog
```
3. Start the daemon `rcctl start pflogd`
4. Edit the `/etc/newsyslog.conf` and replace the command for pflogd with our new rc
```
/var/log/pflog    600  3  250  *  ZB "rcctl reload pflogd"
```

### Create new pflog interface and rc script
1. Create the pflog interface for your next pflogd instance (eg `pflog1`) and activate it.
```shell
foo# echo up > /etc/hostname.pflog1
foo# /etc/netstart pflog1
```

2. Copy the existing `/etc/rc.d/pflogd` file to use as a basis.
```shell
foo# cp /etc/rc.d/pflogd /etc/rc.d/pflogd1
```
3. Edit the file and replace the pflog0 instances with your new interface eg `pflog1`, at the `rc_pre()` function.
```sh
rc_pre() {
	if pfctl -si | grep -q Enabled; then
		ifconfig pflog1 create
		if ifconfig pflog1; then
			ifconfig pflog1 up
		else
			return 1
		fi
	else
		return 1
	fi
}
```

4. Configure your new pflogd1 instance to start on boot and start it
```shell
foo# rcctl enable pflogd1
foo# rcctl set pflogd1 flags -i pflog1 -s 1500 -f /var/log/pflog1
foo# rcctl start pflogd1
```

5. (optional) Update newsyslog.conf to rotate the new log at `/var/log/pflog1`
```shell
/var/log/pflog1				600  3     250  *     ZB "rcctl reload pflogd1"
```

## Final words
The examples here make no "real" or pragmatic sense as they stand. In general the situations that you may need something like this are extremely rare, I would think. Yet if you have to use more than one pflogd and you need to be able to control them individually this is the best way to go.
