## More research

### upd(4)

$ sysctl hw.sensors.upd0

```
hw.sensors.upd0.indicator0=On (Charging), OK
hw.sensors.upd0.indicator1=Off (Discharging), OK
hw.sensors.upd0.indicator2=Off (NeedReplacement), OK
hw.sensors.upd0.indicator3=Off (ShutdownImminent), OK
hw.sensors.upd0.indicator4=On (ACPresent), OK
hw.sensors.upd0.indicator5=Off (Overload), OK
hw.sensors.upd0.percent0=100.00% (RemainingCapacity), OK
hw.sensors.upd0.percent1=100.00% (FullChargeCapacity), OK
hw.sensors.upd0.timedelta0=0.000000 secs (RunTimeToEmpty), OK
```

$ zgrep sensorsd /⁠var/⁠log/⁠daemon*

```
/var/log/daemon:Nov  4 07:00:20 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon:Nov  4 08:58:24 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon:Nov  4 08:58:44 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon:Nov  4 09:31:37 server sensorsd[1790]: startup, system has 40 sensors
/var/log/daemon:Nov  4 09:31:52 server sensorsd[10211]: upd0.indicator0: On, OK
/var/log/daemon:Nov  4 09:31:52 server sensorsd[10211]: upd0.indicator1: Off, OK
/var/log/daemon:Nov  4 09:31:52 server sensorsd[10211]: upd0.indicator2: Off, OK
/var/log/daemon:Nov  4 09:31:52 server sensorsd[10211]: upd0.indicator3: Off, OK
/var/log/daemon:Nov  4 09:31:52 server sensorsd[10211]: upd0.indicator4: On, OK
/var/log/daemon:Nov  4 09:31:52 server sensorsd[10211]: upd0.indicator5: Off, OK
/var/log/daemon:Nov  4 09:31:52 server sensorsd[10211]: upd0.percent0: 100.00%, OK
/var/log/daemon:Nov  4 09:31:52 server sensorsd[10211]: upd0.percent0: within limits: 100.00%
/var/log/daemon:Nov  4 09:31:52 server sensorsd[10211]: upd0.percent1: 100.00%, OK
/var/log/daemon:Nov  4 09:31:52 server sensorsd[10211]: upd0.timedelta0: 0.000000 secs, OK
/var/log/daemon:Nov  4 09:31:52 server sensorsd[10211]: softraid0.drive0: online, OK
/var/log/daemon:Nov  4 09:32:31 server sensorsd[15990]: startup, system has 40 sensors
/var/log/daemon:Nov  4 09:32:46 server sensorsd[15230]: upd0.indicator0: On, OK
/var/log/daemon:Nov  4 09:32:46 server sensorsd[15230]: upd0.indicator1: Off, OK
/var/log/daemon:Nov  4 09:32:46 server sensorsd[15230]: upd0.indicator2: Off, OK
/var/log/daemon:Nov  4 09:32:46 server sensorsd[15230]: upd0.indicator3: Off, OK
/var/log/daemon:Nov  4 09:32:46 server sensorsd[15230]: upd0.indicator4: On, OK
/var/log/daemon:Nov  4 09:32:46 server sensorsd[15230]: upd0.indicator5: Off, OK
/var/log/daemon:Nov  4 09:32:46 server sensorsd[15230]: upd0.percent0: 100.00%, OK
/var/log/daemon:Nov  4 09:32:46 server sensorsd[15230]: upd0.percent0: within limits: 100.00%
/var/log/daemon:Nov  4 09:32:46 server sensorsd[15230]: upd0.percent1: 100.00%, OK
/var/log/daemon:Nov  4 09:32:46 server sensorsd[15230]: upd0.timedelta0: 0.000000 secs, OK
/var/log/daemon:Nov  4 09:32:46 server sensorsd[15230]: softraid0.drive0: online, OK
/var/log/daemon.0.gz:Nov  3 21:47:35 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.0.gz:Nov  3 21:47:55 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.0.gz:Nov  3 22:48:57 server sensorsd[5]: upd0.indicator0: On, UNKNOWN
/var/log/daemon.0.gz:Nov  3 22:48:57 server sensorsd[5]: upd0.indicator1: Off, UNKNOWN
/var/log/daemon.0.gz:Nov  3 22:48:57 server sensorsd[5]: upd0.indicator2: Off, UNKNOWN
/var/log/daemon.0.gz:Nov  3 22:48:57 server sensorsd[5]: upd0.indicator3: Off, UNKNOWN
/var/log/daemon.0.gz:Nov  3 22:48:57 server sensorsd[5]: upd0.indicator4: On, UNKNOWN
/var/log/daemon.0.gz:Nov  3 22:48:57 server sensorsd[5]: upd0.indicator5: Off, UNKNOWN
/var/log/daemon.0.gz:Nov  3 22:49:17 server sensorsd[5]: upd0.indicator0: On, OK
/var/log/daemon.0.gz:Nov  3 22:49:17 server sensorsd[5]: upd0.indicator1: Off, OK
/var/log/daemon.0.gz:Nov  3 22:49:17 server sensorsd[5]: upd0.indicator2: Off, OK
/var/log/daemon.0.gz:Nov  3 22:49:17 server sensorsd[5]: upd0.indicator3: Off, OK
/var/log/daemon.0.gz:Nov  3 22:49:17 server sensorsd[5]: upd0.indicator4: On, OK
/var/log/daemon.0.gz:Nov  3 22:49:17 server sensorsd[5]: upd0.indicator5: Off, OK
/var/log/daemon.0.gz:Nov  3 23:49:24 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.0.gz:Nov  3 23:49:44 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.0.gz:Nov  4 03:19:59 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.0.gz:Nov  4 03:20:19 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.0.gz:Nov  4 05:01:01 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.0.gz:Nov  4 05:01:21 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.0.gz:Nov  4 06:13:14 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.0.gz:Nov  4 06:13:34 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.0.gz:Nov  4 06:53:04 server sensorsd[5]: upd0.indicator2: Off, UNKNOWN
/var/log/daemon.0.gz:Nov  4 06:53:04 server sensorsd[5]: upd0.indicator3: Off, UNKNOWN
/var/log/daemon.0.gz:Nov  4 06:53:04 server sensorsd[5]: upd0.indicator5: Off, UNKNOWN
/var/log/daemon.0.gz:Nov  4 06:53:24 server sensorsd[5]: upd0.indicator2: Off, OK
/var/log/daemon.0.gz:Nov  4 06:53:24 server sensorsd[5]: upd0.indicator3: Off, OK
/var/log/daemon.0.gz:Nov  4 06:53:24 server sensorsd[5]: upd0.indicator5: Off, OK
/var/log/daemon.0.gz:Nov  4 07:00:00 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.1.gz:Nov  3 13:36:52 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.1.gz:Nov  3 13:37:12 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.1.gz:Nov  3 14:30:33 server sensorsd[5]: upd0.indicator0: On, UNKNOWN
/var/log/daemon.1.gz:Nov  3 14:30:33 server sensorsd[5]: upd0.indicator1: Off, UNKNOWN
/var/log/daemon.1.gz:Nov  3 14:30:33 server sensorsd[5]: upd0.indicator4: On, UNKNOWN
/var/log/daemon.1.gz:Nov  3 14:30:38 server sensorsd[5]: upd0.percent0: 100.00%, UNKNOWN
/var/log/daemon.1.gz:Nov  3 14:30:38 server sensorsd[5]: upd0.timedelta0: 0.000000 secs, UNKNOWN
/var/log/daemon.1.gz:Nov  3 14:30:58 server sensorsd[5]: upd0.indicator0: On, OK
/var/log/daemon.1.gz:Nov  3 14:30:58 server sensorsd[5]: upd0.indicator1: Off, OK
/var/log/daemon.1.gz:Nov  3 14:30:58 server sensorsd[5]: upd0.indicator4: On, OK
/var/log/daemon.1.gz:Nov  3 14:30:58 server sensorsd[5]: upd0.percent0: 100.00%, OK
/var/log/daemon.1.gz:Nov  3 14:30:58 server sensorsd[5]: upd0.timedelta0: 0.000000 secs, OK
/var/log/daemon.1.gz:Nov  3 15:34:16 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.1.gz:Nov  3 15:34:36 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.1.gz:Nov  3 16:24:02 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.1.gz:Nov  3 16:24:22 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.1.gz:Nov  3 17:41:46 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.1.gz:Nov  3 17:42:06 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.2.gz:Nov  3 08:37:22 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.2.gz:Nov  3 08:37:47 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.2.gz:Nov  3 12:07:22 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.2.gz:Nov  3 12:07:47 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.3.gz:Nov  2 20:03:03 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.3.gz:Nov  2 20:03:23 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.3.gz:Nov  2 21:37:34 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.3.gz:Nov  2 21:37:54 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.3.gz:Nov  2 21:45:05 server sensorsd[5]: upd0.indicator2: Off, UNKNOWN
/var/log/daemon.3.gz:Nov  2 21:45:05 server sensorsd[5]: upd0.indicator3: Off, UNKNOWN
/var/log/daemon.3.gz:Nov  2 21:45:05 server sensorsd[5]: upd0.indicator5: Off, UNKNOWN
/var/log/daemon.3.gz:Nov  2 21:45:30 server sensorsd[5]: upd0.indicator2: Off, OK
/var/log/daemon.3.gz:Nov  2 21:45:30 server sensorsd[5]: upd0.indicator3: Off, OK
/var/log/daemon.3.gz:Nov  2 21:45:30 server sensorsd[5]: upd0.indicator5: Off, OK
/var/log/daemon.3.gz:Nov  3 00:33:35 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.3.gz:Nov  3 00:34:00 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.3.gz:Nov  3 01:14:25 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.3.gz:Nov  3 01:14:45 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.3.gz:Nov  3 02:41:55 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.3.gz:Nov  3 02:42:15 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.3.gz:Nov  3 04:27:43 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.3.gz:Nov  3 04:28:03 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.3.gz:Nov  3 06:07:39 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.3.gz:Nov  3 06:07:59 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.3.gz:Nov  3 07:49:16 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.3.gz:Nov  3 07:49:41 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.4.gz:Nov  2 06:40:09 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.4.gz:Nov  2 06:40:29 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.4.gz:Nov  2 09:53:57 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.4.gz:Nov  2 09:54:17 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.4.gz:Nov  2 12:29:05 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.4.gz:Nov  2 12:29:25 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.4.gz:Nov  2 13:50:24 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.4.gz:Nov  2 13:50:49 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.4.gz:Nov  2 17:34:41 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.4.gz:Nov  2 17:35:01 server sensorsd[5]: upd0.percent0: within limits: 100.00%
/var/log/daemon.4.gz:Nov  2 18:04:34 server sensorsd[5]: upd0.percent0: exceeds limits: 19.00% is below 20.00%
/var/log/daemon.4.gz:Nov  2 18:04:54 server sensorsd[5]: upd0.percent0: within limits: 100.00%
```

$ cat /⁠etc/⁠sensorsd.conf
```
hw.sensors.upd0.percent0:low=20%:command=/⁠etc/⁠sensorsd/⁠ups.sh shutdown
hw.sensors.upd0.indicator0:command=/⁠etc/⁠sensorsd/⁠ups.sh charging
hw.sensors.upd0.indicator1:command=/⁠etc/⁠sensorsd/⁠ups.sh discharging
```
