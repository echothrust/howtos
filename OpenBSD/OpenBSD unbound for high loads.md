# OpenBSD unbound for high loads
The following document is sniped from the misc mailing list of OpenBSD that can be found at:
* https://misc.openbsd.narkive.com/sewRPMyC/unbound-eats-up-buffer-space

On a gateway with unbound as a resolver for a LAN we're seeing these in our log:

```
Mar  8 08:21:42 kerber unbound: [24074:0] notice: sendto failed: No buffer space available
Mar  8 08:21:42 kerber unbound: [24074:0] notice: remote address is 192.168.33.1 port 60829
```

Give unbound more file descriptors; put in `login.conf`:

```
unbound:\
    :openfiles=512:\
    :tc=daemon:
```

Unbound has a non-standard `setrlimit` setup that requires and explicit login class. Set the user `_unbound` class to `unbound` and verify:

```
$ userinfo _unbound
login   _unbound
passwd  *
uid     601
groups  _unbound
change  NEVER
class   unbound   <--------
gecos   Unbound Daemon
dir     /var/unbound
shell   /sbin/nologin
expire  NEVER
```

If pf queues are in used these error can happen when there's no space left in a queue.
Running `pfctl -v -s queue` you will notice `net.inet.udp.sendspace` to be raised.

The following options for `unbound.conf` may be also useful on on busy servers

```
num-⁠threads
outgoing-⁠range
num-⁠queries-⁠per-⁠thread
```
