---
tags:
  - WIP
  - research
  - OpenBSD
  - mtree
  - tripwire
---

OpenBSD already does an excellent work at keeping track of system changes by `security(8)`  (`/usr/libexec/security`)


```shell
# mtree -cx -p /bin -K sha256digest,type > /etc/mtree/bin.secure
# chown root:wheel /etc/mtree/bin.secure
# chmod 600 /etc/mtree/bin.secure
```
