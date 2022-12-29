# Courier IMAP Tips and Tricks
## Create Courier IMAP shared folders & subfolders
  * Create the Maildir that will hold the shared content `maildirmake -S /etc/shared/OpenBSD`
  * Create the shared Maildir Folder that will hold the mails `maildirmake -s write -f IPv6 /etc/shared/OpenBSD`
  * Change ownerships, groups and mode `chown -R vmail._courier /etc/shared/OpenBSD`
  * Add entry into `/etc/courier/maildirshared` `echo -e "IPv6\t/etc/shared/OpenBSD/.IPv6\n">>/etc/courier/maildirshared`
  * Restart ''imapd'' and ''imapd-ssl''
```
/usr/local/libexec/imapd.rc restart
/usr/local/libexec/imapd-ssl.rc restart
```

In order to create shared subfolders into an existing shared folder we use the following format using the dot '.'

<code>
# maildirmake -s write -f IPv6.subIPv6 /etc/shared/OpenBSD
</code>
