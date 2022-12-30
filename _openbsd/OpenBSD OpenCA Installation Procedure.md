---
---

# OpenBSD OpenCA Installation Procedure
## Introduction
The following document outlines the installation instructions for OpenCA under OpenBSD.

## Configuration
The following installation was performed under OpenBSD 3.8 although the process should apply to newer versions.

```bash
./configure --prefix=/opt --with-openca-prefix=/opt \
 --with-httpd-user=www --with-httpd-group=www \
 --with-openca-user=openca --with-opeca-group=openca \
 --with-httpd-fs-prefix=/var/www \
 --with-web-host=openca.echothrust.dev
gmake
mkdir -p /opt/etc/{servers,access_control}
gmake install-common install-online install-offline install-ca install-pub
```

After the installation is over a good thing to do is replace the OpenCA Perl XML-Twig package
with the default package from the openbsd ports as for some reason the original seems to fail...
pkg_add ftp://ftp.openbsd.org/pub/OpenBSD/3.8/packages/i386/p5-XML-Twig-3.15.tar.gz

Now if everything is well
```bash
cp src/common/etc/config.xml /opt/etc/
cd /opt/
sh etc/configure_etc.sh
```
