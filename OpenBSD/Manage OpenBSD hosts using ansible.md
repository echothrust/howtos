# Manage OpenBSD hosts using ansible

## First steps with ansible

All Ansible commands follow the pattern:

```
ansible <server_or_group> -m module_name -a arguments
```

Run ansible test on OpenBSD host (requires python2.7 installed):

```
ansible all -u sysadmin -i www.echoctf.dev, -m ping -e 'ansible_python_interpreter=/usr/local/bin/python2.7'
```

## Ansible host inventory

Create inventory location:

```
mkdir ~/work/ansible
touch ~/work/ansible/hosts
touch ~/.ansible.cfg
```

Open `~/.ansible.cfg` file to specify the inventory location:

```
[defaults]
inventory = ~/work/ansible/hosts
```

Create entries in `~.work/ansible/hosts` file:

```
kerberus.wks.echothrust.dev
mail.echothrust.dev

[webservers]
www.echoctf.dev
support.echothrust.dev
www.echothrust.dev
```

## Creating playbooks

A playbook is a YAML file, and typically follows this structure:

```
---
- hosts: [target hosts]
  remote_user: [yourname]
  tasks:
    - [task 1]
    - [task 2]
```

For example, the following playbook will create a file on all servers in the `webservers` group

```
---
- hosts: [webservers]
  remote_user: sysadmin
  tasks:
    - name: Create /tmp/somefile.test
      command: touch /tmp/somefile.test
      become: True
      become_method: doas
```

Relevant [post about doas, ansible and env vars](https://www.wordspeak.org/posts/making-ansible-doas-and-openbsd-play-nicely.html)


## Running playbooks

Assuming you are in the same directory as a playbook file, run:

```
ansible-playbook myplaybook.yml
```

If you want to see what hosts this playbook will affect without having to open up the YAML file, you can run:

```
ansible-playbook myplaybook.yml --list-hosts
```

If you want to see what tasks will run on a specific host:

```
ansible-playbook myplaybook.yml -i www.echoctf.dev, --list-tasks
```

## Use the "batteries included"

Ansible ships with a large collection of modules that you can run as tasks or via ad-hoc commands. To see a listing of all available modules, run:

```
ansible-doc -l
```

The list is quite large... some interesting modules follow.

Commands:

 * [command](http://docs.ansible.com/ansible/command_module.html) - Executes a command on a remote node
 * [script](http://docs.ansible.com/ansible/script_module.html) - Runs a local script on a remote node after transferring it
 * [shell](http://docs.ansible.com/ansible/shell_module.html) - Execute commands in nodes
 * [raw](http://docs.ansible.com/ansible/raw_module.html) - Executes a low-down and dirty SSH command
 * [fetch](http://docs.ansible.com/ansible/fetch_module.html) - Fetches a file from remote nodes

Files:

 * [copy](http://docs.ansible.com/ansible/copy_module.html) - Copies files to remote locations
 * [template](http://docs.ansible.com/ansible/template_module.html) - Templates a file out to a remote server
 * [authorized_key](http://docs.ansible.com/ansible/authorized_key_module.html) - Add/remove SSH authorized keys
 * [known_hosts](http://docs.ansible.com/ansible/known_hosts_module.html) - Add or remove a host from the `known_hosts` file
 * [lineinfile](http://docs.ansible.com/ansible/lineinfile_module.html) - Ensure a particular line is in a file. Replace existing line using a back-referenced regex
 * [blockinfile](http://docs.ansible.com/ansible/blockinfile_module.html) - Insert/update/remove a text block surrounded by marker lines
 * [replace](http://docs.ansible.com/ansible/replace_module.html) - Replace all instances of a particular string in a file using a back-referenced regular expression
 * [ini_file](http://docs.ansible.com/ansible/ini_file_module.html) - Tweak settings in INI files
 * [htpasswd](http://docs.ansible.com/ansible/htpasswd_module.html) - Manage user files for basic authentication
 * [stat](http://docs.ansible.com/ansible/stat_module.html) - Retrieve file or file system status
 * [unarchive](http://docs.ansible.com/ansible/unarchive_module.html) - Unpacks an archive after (optionally) copying it from the local machine

Package management:

 * [git](http://docs.ansible.com/ansible/git_module.html) - Deploy software (or files) from git checkouts
 * [openbsd_pkg](http://docs.ansible.com/ansible/openbsd_pkg_module.html) - Manage packages on OpenBSD
 * [yum](http://docs.ansible.com/ansible/yum_module.html) - Manages packages with the yum package manager
 * [apt](http://docs.ansible.com/ansible/apt_module.html) - Manages apt-packages


Operating system:

 * [service](http://docs.ansible.com/ansible/service_module.html) - Manage services
 * [system](http://docs.ansible.com/ansible/systemd_module.html) - Manage services
 * [user](http://docs.ansible.com/ansible/user_module.html) - Manage user accounts
 * [cron](http://docs.ansible.com/ansible/cron_module.html) - Manage cron.d and crontab entries.
 * [solaris_zone](http://docs.ansible.com/ansible/solaris_zone_module.html) - Manage Solaris zones
 * [sysctl](http://docs.ansible.com/ansible/sysctl_module.html) - Manage entries in sysctl.conf.

Various:

 * [mysql_db](http://docs.ansible.com/ansible/mysql_db_module.html) - Add or remove MySQL databases from a remote host
 * [mysql_user](http://docs.ansible.com/ansible/mysql_user_module.html) - Adds or removes a user from a MySQL database
 * [nagios](http://docs.ansible.com/ansible/nagios_module.html) - Perform common tasks in Nagios related to downtime and notifications
 * [redis](http://docs.ansible.com/ansible/redis_module.html) - Various redis commands, slave and flush
 * [letsencrypt](http://docs.ansible.com/ansible/letsencrypt_module.html) - Create SSL certificates with Let's Encrypt
 * [cloudflare_dns](http://docs.ansible.com/ansible/cloudflare_dns_module.html) - manage Cloudflare DNS records
 * [digital_ocean](http://docs.ansible.com/ansible/digital_ocean_module.html) - Create/delete a droplet/SSH_key in DigitalOcean
 * [wakeonlan](http://docs.ansible.com/ansible/wakeonlan_module.html) - Send a magic Wake-on-LAN (WoL) broadcast packet
