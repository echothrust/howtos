---
---

# Oracle Grid Background Processes
All Oracle background processes for Oracle RAC 10g are listed below. Some of them are not required background processes, which mean than they may not run in the system.

^ Process Name ^ Description ^
| pmon | process monitor & cleanup |
| diag | diagnosibility process |
| fmon | file mapping monitor process |
| psp0 | process spawner 0 |
| lmon | global enqueue service monitor |
| lmd0 | global enqueue service monitor deamon 0 |
| lms{0-9, a-z} | global cache service process from 0-9 and a-z eg: lms6, lmsf |
| mman | memory manager |
| dbw{0-9, a-j} | db writer process 0-9 and a-j |
| arc{0-9, a-t} | archiver process 0-9 and a-t |
| lns{0-9, a-j} | network server (listener process) 0-9 and a-j |
| mrp0 | managed standby recovery |
| lgwr | log writer (redo etc) |
| lck0 | lock process 0 |
| ckpt | checkpoint |
| lsp{0-2} | logical standby process (for standby databases) |
| ctwr | change tracking writer |
| rvwr | recovery writer |
| smon | system monitor |
| reco | distributed recovery |
| cjq0 | job queue monitor |
| emv | event monitor |
| qmnc | AQ coordinator |
| dmon | data guard broker monitor |
| rsm{0-1} | data guard broker resource guard process 0-1 |
| nsv{0-9} | data guard broker netslave process |
| insv | data guard broker instance slave process |
| rbal | asm rebalance master |
| arb{0-9}, arbA | asm rebalance 0-9 and A |
| asmb | asm background |
| gmon | diskgroup monitor |
| mmon | manageability monitor process |
| mmnl | manageability monitor process 2 |
| cssd | css (cluster synchronization service) deamon |
| crsd | crs (cluster ready services) deamon |

There are three main background processes that required for the RAC to be running. You can see when doing a ps –ef|grep d.bin.  They are normally started by init during the operating system boot process.
  - `/etc/rc.d/init.d/init.evmd`
  - `/etc/rc.d/init.d/init.cssd`
  - `/etc/rc.d/init.d/init.crsd`

  $ ps -aef|grep d.bin
  $ /u01/app/oracle/product/10.2.0/crs/bin/evmd.bin
  $ /u01/app/oracle/product/10.2.0/crs/bin/crsd.bin reboot
  $ /u01/app/oracle/product/10.2.0/crs/bin/ocssd.bin

In order to check that the above processes are running correctly you can perform the following command:
  $ crsctl check crs
Once the above processes are running, they will automatically start the following services in the following order if they are enabled.
  - The nodeapps (gsd, VIP, ons, listener) are brought online.
  - The ASM instances are brought online.
  - The database instances are brought online.
  - Any defined services are brought online.

There are five main background processes that required for the database to be up and running. Those are:
   - smon
   - pmon
   - dbw0 (at least)
   - lgwr
   - ckpt