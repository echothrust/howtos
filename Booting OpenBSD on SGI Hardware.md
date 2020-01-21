# Notes on Booting OpenBSD on SGI Hardware

## SGI O2 Preparation =====
In order to start the installation from the CD you need to enter the following
commands onto the `maintenance` mode of your O2.
```
>> resetenv
>> setenv OSLoader boot
>> setenv OSLoadFilename /bsd.rd
>> setenv console d
```

After the installation is finished you need to change the `OSLoadFilename` from
the maintenance console to the normal kernel `bsd` instead of one used for the
installation `bsd.rd`
```
>> setenv OSLoadFilename /bsd
```
{{tag>OpenBSD booting SGI 02}}
