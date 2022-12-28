# General things to know about Putty
Author: Dimitris Zoakos <dzoakos@echothrust.com>
Date: 2010
Putty is a suite for remote communications in a variety of options (SSH, Telnet, Serial). For further information browse the [[http://www.chiark.greenend.org.uk/~sgtatham/putty/|home page]] of the software.

## Generation and usage of Keys with Putty
We assume that we don't have the putty in our machine. So, first step is to
download the http://tartarus.org/~simon/putty-snapshots/x86/putty-installer.exe and fully install the package.

When the installation is completed we can create our pair of keys (private and
public).
For this reason we run the puttygen.exe from the Putty installation folder.
We click on 'Generate' button and we see the following window
{{:users:dzoakos:notes:putty_gen.png|Putty Generator}}

Then, we move our mouse cursor inside the blank area right bellow the progress bar. When the key is generated we have to save the private key. The extension of the filetype is .ppk. We have also the option to save as plaintext the public part of our key.

Putty Client, in order to know who we are, we will have to load for one time our key. To do that we have to run the pageant.exe. This application will run minimized on tray icon. We locate it, right-click it and select 'Add key' option. We browse to our private copy of our key (filename.ppk) and load it. If we use a passphrase (which is the right thing to do... ;-)) it will ask us to type it. After that our pair of keys is loaded and are available for Putty Client.

Now we can use our keys to connect without a password on every account that has our public key in authorized_keys file. To do that we run the putty.exe and type the hostname of the machine we want to connect to. The difference is that we go to Connection->SSH->Auth as bellow and we select the 'Allow agent forwardin' and we browse for our filename.ppk.

{{:users:dzoakos:notes:putty_client_auth.png|}}

As we click on 'Open' button we should connect to host.

## Exporting Private Key from .ppk file
In order to export our private key from our .ppk file,
* we can run the puttugen.exe
* we load our filename.ppk by pressing the 'Load' button
* then go to Conversions->Export OpenSSH key
* choose a file name to be saved
... and voila, we have a plaintext file with our private key, ready to use.
