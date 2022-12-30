---
tags:
  - OpenBSD
  - Gnome
  - Xfce
  - Firefox
  - VirtualBox
  - Vesa
---

# Doing it 1999 style: OpenBSD Gnome + Xfce + Firefox on 16GB VirtualBox VM
What do they say? Actions speak louder than a thousand words? I like it.

Fair warning, this must be the most unnecessary howto in the history of howtos but there you have it 😂

Everything mentioned on this guide, can be found on the FAQ, manual pages and readme documents of the applications installed. Again this is mostly towards new users who dont even know where to start to install a desktop manager that is more friendly to them.

## The premise
So you've just finished installing your OpenBSD and want to install a manager you're more familiar with but you dont have a lot of space for this system... 

Someone once told me: _you shouldn't be expecting to run any modern desktop on just 16GB._ 🤮 and maybe in parts is right, but in this case is not.

Not to worry though. We'll go through this process together and hopefully get you up and running with a more modern desktop on just under 16GB.

## Before we start
Now you may have guessed by the title, this guide is primarily for these two window managers, however, you can use the same methods described to find out what is needed and install any manager and application you like from the OpenBSD packages.

One of the 1st things I like to do before I start installing packages is to fetch the text version of the packages index and use it to look for the package names that may interest me. If you dont know where to get it, check the `/etc/installurl` contents. In the following example it points to the local OpenBSD mirror (neat!)
```shell
foo# cat /etc/installurl
https://ftp.cc.uoc.gr/pub/OpenBSD
```

The OpenBSD mirror URLs follow these rules:
* `MAJOR.MINOR/`: source code files to save time when checking out updated code
* `MAJOR.MINOR/ARCHITECTURE`: Version release files/installation media
* `MAJOR.MINOR/packages/ARCHITECTURE`: Packages for a release

Following the last entry for the packages, we will fetch a file called `index.txt` which includes a list (`ls`) of all packages in text format.

So lets grab it (both of the ftp commands are valid, one will only work for OpenBSD 7.1 and the other for most other versions)
```shell
foo# ftp https://ftp.cc.uoc.gr/pub/OpenBSD/7.1/packages/amd64/index.txt
                                           ^^^         ^^^^^^
                                        major.minor   architecture
foo# ftp https://ftp.cc.uoc.gr/pub/OpenBSD/$(uname -r)/packages/$(uname -m)/index.txt
Trying 147.52.159.12...
Requesting https://ftp.cc.uoc.gr/pub/OpenBSD/6.5/packages/amd64/index.txt
100% |*************************************************************|   787 KB    00:00    
806431 bytes received in 0.46 seconds (1.67 MB/s)
```

## Installing the software
Lets see if we can find the managers we want. Notice that we use the **`-i`** to disable case sensitive matching, since we dont know if the package we are looking for is with capital letters or not.
```shell
foo# grep -i firefox index.txt
foo# grep -i gnome index.txt
foo# grep -i xfce index.txt
```

WOW! It seems that there are a lot of gnome and xfce packages there. Should we pick one or all of them? Which one?



A more `i'm feeling lucky` approach is to simply try and install the software you're looking for and (🤞) see what you get
```shell
foo# pkg_add -vi firefox gnome xfce
```

