---
---

# Symon install and config

## OpenBSD
The package `simon-<>.tgz` contains both client and server requirements.

### Server
In order to set up an openbsd as a symon info gatherer we have to install the
`symon-mux-2.85.tgz`.

After that we have to edit and configure `/etc/symux.conf` by adding the clients like the example below

```
#
# Demo symux configuration. See symux(8) for BNF.
#

# ip:port of our server
mux 172.16.10.252 2100

# Define each source and it's data.
#
source 172.16.10.254 {
        accept { cpu(0),mem,mbuf,io(wd0),io(wd1),if(em0),if(em1),if(em2),if(em3),if(em4),if(nfe0),if(pppoe0),pf }
        write cpu(0)   in "/var/www/symon/rrds/kerberus/cpu0.rrd"
        write mem      in "/var/www/symon/rrds/kerberus/mem.rrd"
        write mbuf     in "/var/www/symon/rrds/kerberus/mbuf.rrd"
        write pf in "/var/www/symon/rrds/kerberus/pf.rrd"
        write if(em0) in "/var/www/symon/rrds/kerberus/if_em0.rrd"
        write if(em1) in "/var/www/symon/rrds/kerberus/if_em1.rrd"
        write if(em2) in "/var/www/symon/rrds/kerberus/if_em2.rrd"
        write if(em3) in "/var/www/symon/rrds/kerberus/if_em3.rrd"
        write if(em4) in "/var/www/symon/rrds/kerberus/if_em4.rrd"
        write if(nfe0) in "/var/www/symon/rrds/kerberus/if_nfe0.rrd"
        write if(pppoe0) in "/var/www/symon/rrds/kerberus/if_pppoe0.rrd"
        write io(wd0)  in "/var/www/symon/rrds/kerberus/io_wd0.rrd"
        write io(wd1)  in "/var/www/symon/rrds/kerberus/io_wd1.rrd"
}
```

When we complete the addition of the clients, we must create the target
directories for rrd files and create the initial files.

In the example's case we should do the following for each client.
```sh
mkdir -p /var/www/symon/rrds/<host>
cd /var/www/symon/rrds
/usr/local/share/symon/c_smrrds.sh all
```

After that we can check that the configuration file is OK by running symux with -t option
```sh
/usr/local/libexec/symux -t
/etc/symux.conf: ok
```

And now we can start the symux server
```sh
/etc/rc.d/symux start
```

### Client
In order to install client of symon we must install the `symon-mon-2.85.tgz`.

The configuration file is located on `/etc/symon.conf`
