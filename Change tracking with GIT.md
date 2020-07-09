## Server preparation
Example with `gw/gw.example.com` replace with hostname and fqdn of rsyncsrv server name.

```sh
export FQDN=gw.example.com
export GIT_DIR=/data/gitrsyncsrv/${FQDN}/
export GIT_WORK_TREE=/data/rsyncsrv/${FQDN}/
export GIT_AUTHOR_EMAIL="backups@example.com"
export GIT_COMMITTER_EMAIL="backups@example.com"
export GIT_AUTHOR_NAME="Backups User @ GW"
export GIT_COMMITTER_NAME="Backups User @ GW"

mkdir -p ${GIT_DIR}
cd ${GIT_WORK_TREE}
#git init
#  or
git --git-dir=${GIT_DIR} --work-tree=${GIT_WORK_TREE} init

cat >${GIT_DIR}/info/exclude <<__EOF__
home/etsbackup/service_dumps/
etc/rmt
etc/resolv.conf.tail
etc/termcap
etc/localtime
etc/X11/
etc/examples/
etc/fonts/
etc/hotplug/
etc/firmware/
etc/sliphome/
etc/random.seed
__EOF__

cd ${GIT_WORK_TREE}
sudo find etc root home var -type f -a \( -iname "*.old" -o -iname "*.db" -o -iname "*.tdb" -o -iname "*.orig" -o -iname "*.sample" \) >> ${GIT_DIR}/info/exclude
sudo find etc root home var -type f -a \( -iname "*.tar" -o -iname "*.tgz" -o -iname "*.gz" \) >> ${GIT_DIR}/info/exclude
cd ${GIT_DIR}
sudo git add etc
sudo git add var
sudo git add root
sudo git add home
sudo git commit
```

* unset all exports once finished
```
unset GIT_AUTHOR_NAME GIT_COMMITTER_NAME GIT_DIR GIT_WORK_TREE etc
```

## Client configuration
```
sudo su -
export GIT_DIR=/altroot/
export GIT_WORK_TREE=/
export GIT_AUTHOR_EMAIL="backups@example.com"
export GIT_COMMITTER_EMAIL="backups@example.com"
export GIT_AUTHOR_NAME="Backups User @ GW"
export GIT_COMMITTER_NAME="Backups User @ GW"

mkdir -p ${GIT_DIR}
cd ${GIT_WORK_TREE}
git init


cat >${GIT_DIR}/info/exclude <<__EOF__
altroot/
bin/
dev/
mnt/
sbin/
sys
tmp/
usr/
var/
home/etsbackup/service_dumps/
etc/master.passwd
etc/rmt
etc/resolv.conf.tail
etc/termcap
etc/localtime
etc/X11/
etc/examples/
etc/fonts/
etc/hotplug/
etc/firmware/
etc/sliphome/
etc/random.seed
__EOF__

cd ${GIT_DIR}
sudo git commit -a
```
