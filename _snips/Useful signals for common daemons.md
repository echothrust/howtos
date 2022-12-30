---
---

# Useful signals for common daemons
Useful signals for some common applications and what they do

## nginx
```
TERM, INT fast shutdown
QUIT graceful shutdown
HUP changing configuration, keeping up with a changed time zone (only for FreeBSD and Linux), starting new worker processes with a new configuration, graceful shutdown of old worker processes
USR1 re-opening log files
USR2 upgrading an executable file
WINCH graceful shutdown of worker processes
```

## openvpn
```
SIGUSR1 Conditional restart
SIGHUP Hard restart
```

## php-fpm
```
SIGINT,SIGTERM immediate termination
SIGQUIT graceful stop
SIGUSR1 re-open log file
SIGUSR2 graceful reload of all workers + reload of fpm conf/binary
```
