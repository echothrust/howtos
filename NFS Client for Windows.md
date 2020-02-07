# NFS Client for Windows
**Author:** Leontis Martopoulos\
**Created:** 2006/07/30 20:54

## Introduction
There are a number of Commercial NFS clients on Windows, but for
cost/licensing issues I have decided to proceed with the SFU 3.5 from Microsoft.

Besides being free, it has the advantage of being part of Windows and not a
third party application, therefore integration with the system is better.

The only drawback is that the download size is around ~250 MB. Included in the
SFU though are a great number of UNIX tools that can also help in future tasks
that have to do with Windows/Unix integration.

## Installing SFU
The following are the steps required for the Setup of a NFS client on Windows
in order to access and map NFS shares from Unix servers in Windows clients:

  * Download SFU 3.5 from Microsoft ([[http://www.microsoft.com/downloads/details.aspx?FamilyID=896c9688-601b-44f1-81a4-02878ff11778&DisplayLang=en|External Free Download]]). Registration Required.
  * Run Setup, chose default installation.
  * Restart Windows PC (client)

## Configuring

* Go to Administrative Tools/Services and find the User Name Mapping Service.
   - Change startup mode to Automatic.
* Obtain the passwd/group files from the Unix NFS Server.
* Copy them to `C:\SFU\common`
* Go to Windows Services for Unix/Services for Unix Administration.
* Chose User Name Mapping.
* From the Configuration tab select Use Password and Group Files.
* Give the location of the passwd/group file from the browse buttons.
* From the Maps tab select Simple Maps and click Show User Maps (make sure the client PC is selected from the WINDOWS DOMAIN NAME list)
* Click Show User Maps. Then List Window Users and List UNIX Users
* Select a user from the Windows users lists and the user from Unix list that you want to map to, e.g Administrator from Windows and Root from Unix.
* Click Add and accept any warning you might see on screen.
* The mapping will appear on the Mapped users list.
* Press Apply
* On the Maps screen click on the Show Group Maps in case you want to map windows groups to Unix Groups.
  - The procedure is the same as above for the users mapping.
* Make sure the User Name Mapping service is started, if not right click the User Name Mapping and choose Start.
* Restart Windows

## Preparing the Server OpenBSD
See OpenBSD NFS Howto.

## Preparing the Server Linux
Setup the NFS server on the UNIX server. NFS v3 is required for SFU 3.5,
although Microsoft claims that NFS v2 is also supported, there are problems
with it.

Make sure the NFS is running.

Run the `rpcinfo -p` command and make sure you see a list containing at least `portmapper`, `nfs` and `mountd`.

Edit the `/etc/exports` file and put the directories that you want to share in the following format:
```
/directory_to_be_shared NFS_client_IP(options)
```
Example:
```
/var/log        192.168.1.2(rw,async,no_root_squash)
```

Save the file and run `exportfs -avr`. Remember to run this command every time
you edit the exports file.

Restart the NFS daemons, there are some cases where that might not work and you
will have to restart the system.

## Mounting
In order to mount a remote NFS export follow the steps:
   * Go to the windows machine where you have installed the NFS client.
    - Chose My Network Places/Entire Network(entire contents).
    - A new NFS Network is available after the installation of SFU 3.5
   * Select the Default LAN.
    - You should be able to see the Unix NFS server and after opening it you should be able to see a list of shares that you have exported.
   * Right Click a share and choose Map Network Drive.
   * Select a Drive Letter, check Reconnect at logon and press Finish.
     - A window will pop up where you will see the user that you are logged in as on the windows pc and it will ask you if you want to log as that user in the NFS server.
     - Choose yes to accept or No to change the user login settings.

After that you will be able to see the NFS shared folder as a drive in My Computer.
