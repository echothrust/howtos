---
---

# Sync Hardware clock on OpenBSD
When you start getting a lot of messages like those, it is usually a sign that
the hardware clock is starting to loose its sync with the ntp daemon.

```
Jan 31 10:15:31 gateway ntpd[20060]: adjusting local clock by 93.846882s
```

In order to correct this problem and avoid flooding your syslog messages you
can try one of the following
  * halt or reboot the machine in order for boot(9) to sync the RTC
  * Set the clock once by hand (on bios)
  * use rdate once and then ntpd

> When the system is booted into multi-user mode syncing the RTC might cause problems and confuse certain daemons.
