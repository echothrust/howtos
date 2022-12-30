---
---

# General Things To Know about GIT
## Tips
* Get the current branch `git rev-parse --abbrev-ref HEAD`
* Reset repo state `git reset --hard origin/master`



## Create a branch from a tag
* Go to the starting point of the project
```sh
git checkout origin master
```

* fetch all objects
```sh
git fetch origin
```

* Make the branch from the tag
```sh
git branch new_branch tag_name
```

* Checkout the branch
```sh
git checkout new_branch
```

* Push the branch up
```sh
git push origin new_branch
```

A complete example follows
```sh
export TAG=my_tag
export BRANCH=my_new_branch_name
git checkout origin master
git fetch origin
git branch ${BRANCH} ${TAG}
git checkout ${BRANCH}
git push origin ${BRANCH}
```

## Merge 2 repositories with their histories
The following process allows us to merge an SRC repository into DST repository
keeping both repository histories.

```sh
export DST_REPO=ssh://git@gitlab.echothrust.com:6022/php/thytis.git
export SRC_REPO=ssh://git@gitlab.echothrust.com:6022/echothrust/thytis.git
echo "Going to delete src/ dst/ Press any key to continue"
read

rm -rf dst/ src/
git clone ${DST_REPO} dst
git clone ${SRC_REPO} src

cd dst/
git remote add other ../src
git fetch other
git checkout -b PRJMERGE other/master
git commit -m "ReMoved stuff to PRJMERGE"
git checkout master

# should add PRJMERGE/ to master
git merge PRJMERGE
git commit -m "Project Merge"
git remote rm other

# to get rid of the extra branch before pushing
git branch -d PRJMERGE
git push
cd -
```

## Blame
In order to see who commited a change into a particular part of a file the
command `git blame filename` can be used
```
$ git blame authpf.patch
d2f7d2e4 (George Adamopoulos 2015-04-29 13:47:18 +0300  4) retrieving revision 1.123
82e96579 (Pantelis Roditis   2015-05-21 14:57:52 +0300  5) diff -u -r1.123 authpf.c
d2f7d2e4 (George Adamopoulos 2015-04-29 13:47:18 +0300  6) --- usr.sbin/authpf/authpf.c 21 Jan 2015 21:50:32 -0000      1.123
56c93d0a (Pantelis Roditis   2015-06-08 14:28:52 +0300  7) +++ usr.sbin/authpf/authpf.c 8 Jun 2015 11:15:07 -0000
b2a8992f (Pantelis Roditis   2015-05-21 15:08:13 +0300  8) @@ -57,6 +57,7 @@
82e96579 (Pantelis Roditis   2015-05-21 14:57:52 +0300  9)  char        anchorname[PF_ANCHOR_NAME_SIZE] = "authpf";
...
```

## Hooks
git hook to run a command after `git pull` if a specified file was changed.
In this example it's used to run `npm install` if package.json changed and
`bower install` if `bower.json` changed. Run `chmod +x post-merge` to make it
executable then put it into `.git/hooks/`.
```sh
#/usr/bin/env bash
# MIT Β© Sindre Sorhus - sindresorhus.com

# git hook to run a command after `git pull` if a specified file was changed
# Run `chmod +x post-merge` to make it executable then put it into `.git/hooks/`.

changed_files="$(git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD)"

check_run() {
	echo "$changed_files" | grep --quiet "$1" && eval "$2"
}

# Example usage
# In this example it's used to run `npm install` if package.json changed and `bower install` if `bower.json` changed.
check_run package.json "npm install"
check_run bower.json "bower install"

```
