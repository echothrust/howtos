# Ansible Tips and Tricks
---
date: 29/12/2022
---
## Configure your ~/.ssh/config
Ansible provides a lot of ways to manage your ssh connections but it gets messy really fast. The best option to avoid having to troubleshoot names and other such details is to create a `~/.ssh/config` with your inventory host connection details.
Example:
```
Host host10 10.0.0.10
  Hostname 10.0.0.10
  User root
  ForwardAgent yes
  ProxyCommand ssh bastion.host -W %h:%p
```

and then you can run your ansible like so: `ansible -i host10 -m ping`

## Execute module with params
```bash
ansible myhost -m copy -a 'src=/etc/resolv.conf dest=/tmp/resolv.conf owner=root mode=0400'
```

## No inventory invocation
Add the hostname as inventory followed by a **`,`** as well as `-u` for remote user to connect to.
```
ansible -i myhost, -m ping -u remote_user --ask-pass|-k
```
