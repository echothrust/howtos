# Monitoring HDD health status with OpenBSD
A lot of articles have surfaced during the past couple of months with regards
to hard drive health checks and failures. So the following paper is an attempt
to gather all necessary information in order to allow us to check and monitor
the health status of our hard drives and hopefully be able to detect potential
failures before exposing them selves in a more catastrophic manner.


# Toolset
Thankfully OpenBSD comes with utilities that can achieve what we want without
installing much.

The tool that we will use extensively for the tasks is `atactl(8)`, a program
to manipulate ATA (IDE) devices.

Furthermore, the package for the SMART monitoring Tools `smartmontools-x.y.tgz`
is available for OpenBSD if you want to keep your checking scripts consistent
across platforms.

Depending on your hard drive vendor and how closely he follows the SMART
specification it might be a good idea to search for the hardware specs and have
them at hand before you start. This will prove particularly useful when reading
the device exposed attributes.

# Status codes
The following SMART metrics seem to indicate conditions where the disk is about
to die (values taken by the BackBlaze article):

* `SMART 5`: Reallocated Sectors Count Count of reallocated sectors. When the
hard drive finds a read/write/verification error, it marks that sector as
"reallocated" and transfers data to a special reserved area (spare area). This
process is also known as remapping, and reallocated sectors are called "remaps".
The raw value normally represents a count of the bad sectors that have been
found and remapped. Thus, the higher the attribute value, the more sectors the
drive has had to reallocate. This allows a drive with bad sectors to continue
operation; however, a drive which has had any reallocations at all is
significantly more likely to fail in the near future. While primarily used as a
metric of the life expectancy of the drive, this number also affects
performance. As the count of reallocated sectors increases, the read/write
speed tends to become worse because the drive head is forced to seek to the
reserved area whenever a remap is accessed. If sequential access speed is
critical, the remapped sectors can be manually marked as bad blocks in the file
system in order to prevent their use.
* `SMART 187`: Reported Uncorrectable Errors The count of errors that could not
be recovered using hardware ECC (see SMART 195).
* `SMART 188`: Command Timeout The count of aborted operations due to HDD
timeout. Normally this attribute value should be equal to zero and if the value
is far above zero, then most likely there will be some serious problems with
power supply or an oxidized data cable.
* `SMART 197`: Current Pending Sector Count Count of "unstable" sectors
(waiting to be remapped, because of unrecoverable read errors). If an unstable
sector is subsequently read successfully, the sector is remapped and this value
is decreased. Read errors on a sector will not remap the sector immediately
(since the correct value cannot be read and so the value to remap is not known,
and also it might become readable later); instead, the drive firmware remembers
that the sector needs to be remapped, and will remap it the next time it's
written. However some drives will not immediately remap such sectors when
written; instead the drive will first attempt to write to the problem sector and
if the write operation is successful then the sector will be marked good (in
this case, the "Reallocation Event Count" (0xC4) will not be increased). This is
a serious shortcoming, for if such a drive contains marginal sectors that
consistently fail only after some time has passed following a successful write
operation, then the drive will never remap these problem sectors.
* `SMART 198`: Uncorrectable Sector Count or Offline Uncorrectable or Off-Line
Scan Uncorrectable Sector Count The total count of uncorrectable errors when
reading/writing a sector. A rise in the value of this attribute indicates
defects of the disk surface and/or problems in the mechanical subsystem.

It seems that our disks on that server report SMART [5,197,198]

## Read hard drive attributes
```
atactl /dev/sd0c readattr
Attributes table revision: 10
ID      Attribute name                  Threshold       Value   Raw
  1     Raw Read Error Rate               50             95     0x0000006fe42b
  5     Reallocated Sector Count           3            100     0x000000000000
  9     Power-On Hours Count               0             95     0xacda00001396
 12     Device Power Cycle Count           0            100     0x00000000002b
171    *Unknown                            0              0     0x000000000000
172    *Unknown                            0              0     0x000000000000
174    *Unknown                            0              0     0x000000000018
177    *Unknown                            0              0     0x000000000000
181    *Unknown                            0              0     0x000000000000
182    *Unknown                            0              0     0x000000000000
187     Unknown                            0            100     0x000000000000
189     High Fly Writes                    0             40     0x0010002d0028
194     Temperature                        0             40     0x0010002d0028
195     Hardware ECC Recovered             0            120     0x0000006fe42b
196     Reallocation Event Count           3            100     0x000000000000
201     Soft Read Error Rate               0            120     0x0000006fe42b
204     Soft ECC Correction                0            120     0x0000006fe42b
230     GMR Head Amplitude                 0            100     0x000000000064
231     Temperature                       10            100     0x000000000000
233    *Unknown                            0              0     0x000000000824
234    *Unknown                            0              0     0x000000000960
241    *Unknown                            0              0     0x000000000960
242    *Unknown                            0              0     0x0000000000e1
One or more threshold values exceeded!
```


## Identify device characteristics
```
atactl /dev/sd0c identify  
Model: KINGSTON SV300S37A60G, Rev: 527ABBF0, Serial #: 50026B774501EAB0
Device type: ATA, fixed
Cylinders: 16383, heads: 16, sec/track: 63, total sectors: 117231408
Device capabilities:
        ATA standby timer values
        IORDY operation
        IORDY disabling
Device supports the following standards:
ATA-2 ATA-3 ATA-4 ATA-5 ATA-6 ATA-7 ATA-8
Master password revision code 0xfffe
Device supports the following command sets:
        NOP command
        READ BUFFER command
        WRITE BUFFER command
        Host Protected Area feature set
        Read look-ahead
        Write cache
        Power Management feature set
        Security Mode feature set
        SMART feature set
        Flush Cache Ext command
        Flush Cache command
        48bit address feature set
        Set Max security extension commands
        Set Features subcommand required
        Power-up in standby feature set
        Advanced Power Management feature set
        DOWNLOAD MICROCODE command
        IDLE IMMEDIATE with UNLOAD FEATURE
        SMART self-test
        SMART error logging
Device has enabled the following command sets/features:
        NOP command
        READ BUFFER command
        WRITE BUFFER command
        Host Protected Area feature set
        Read look-ahead
        Write cache
        Power Management feature set
        SMART feature set
        Flush Cache Ext command
        Flush Cache command
        48bit address feature set
        Set Features subcommand required
        Advanced Power Management feature set
        DOWNLOAD MICROCODE command
```

## Enable SMART status on the device
```
atactl /dev/sd0c smartenable
```

## Read SMART values
```
atactl /dev/sd0c smartread   
Off-line data collection:
    status: never started
    activity completion time: 0 seconds
    capabilities:
        execute immediate
        read scanning
        self-test routines
Self-test execution:
    status: completed ok or not started
    recommended polling time:
        short routine: 1 minutes
        extended routine: 36 minutes
SMART capabilities:
    saving SMART data
    enable/disable attribute autosave
Error logging: supported
```

## Check SMART status violations
```
atactl /dev/sd0c smartstatus
No SMART threshold exceeded
```



## Begin short device self checks
Takes aproximately 90 seconds
```
atactl /dev/sd0c smartoffline shortoffline
```

While the offline checks are executed on the device the `smartread` will report something like this
```
atactl /dev/sd0c smartread
Off-line data collection:
    status: (null)
    activity completion time: 32 seconds
    capabilities:
        execute immediate
        read scanning
        self-test routines
Self-test execution:
    status: (null)
remains 50% of total time
    recommended polling time:
        short routine: 1 minutes
        extended routine: 36 minutes
SMART capabilities:
    saving SMART data
    enable/disable attribute autosave
Error logging: supported
```


Once the short self-check routine is done the output will look like
```
sudo atactl /dev/sd0c smartread                 
Off-line data collection:
    status: completed ok
    activity completion time: 0 seconds
    capabilities:
        execute immediate
        read scanning
        self-test routines
Self-test execution:
    status: completed ok or not started
    recommended polling time:
        short routine: 1 minutes
        extended routine: 36 minutes
SMART capabilities:
    saving SMART data
    enable/disable attribute autosave
Error logging: supported
```

## Initiate off-line status collection
atactl /dev/sd0c smartoffline collect

# References
* https://www.backblaze.com/blog/hard-drive-smart-stats/
