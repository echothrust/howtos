---
title: Environment variables and how to get them
author: Pantelis Roditis <proditis[at]echothrust.com>
date: 04/06/2021
category: paper
collection: paper
layout: post
---

# Environment variables and how to get them
- [Environment variables and how to get them](#environment-variables-and-how-to-get-them)
  - [Introduction](#introduction)
  - [Why do we care](#why-do-we-care)
  - [How they work](#how-they-work)
    - [docker variables](#docker-variables)
  - [How to get them](#how-to-get-them)
    - [`printenv` - print all or part of environment](#printenv---print-all-or-part-of-environment)
    - [`env` - print the environment or run a program in a modified environment](#env---print-the-environment-or-run-a-program-in-a-modified-environment)
    - [`set` BUILTIN - show or set shell variables and functions](#set-builtin---show-or-set-shell-variables-and-functions)
    - [`declare` / `typeset` BUILTINS - declare variables and/or give them attributes](#declare--typeset-builtins---declare-variables-andor-give-them-attributes)
    - [ps](#ps)
    - [/proc](#proc)
  - [What next](#what-next)
  - [References](#references)

## Introduction
> An **environment variable** is a named value that can be accessed and affect the way running processes will behave on a computer.
> - wikipedia

Think of variables as symbolic names we give to values in order to avoid remembering them. Say you use the number `3.1415926535897932384626433` quite often, and unless you are a robot it is going to be hard to remember the entire number by heart. So you give a name this number `pi=3.1415926535897932384626433` and now whenever you want to use it you simply use the symbolic name `pi`, instead of the huge number sequence.

In this document we will outline the use of environment variables, how they can be accessed, manipulated and taking advantage of them for both offensive and defensive security engineers.

## Why do we care
So why are environment variables important from a security perspective?

It used to be (in the old days lol) that environment variables where visible by everyone and as a rule most developers didn't use them to hold sensitive information.
However, as the adoption of environment variables grew, so was the need to start holding a bit more sensitive information. So the ability to hide the environment variables from other users and processes was added to some, if not all, UNIX and UNIX-like systems (such as Linux) and thus limiting their attack surface.

Now, we've moved into the container era, environment variables got a new meaning. Have a quick look at docker hub and you'll see millions of images that use environment variables that hold sensitive information. From username, passwords, encryption keys, authentication tokens, system keys, there is an image with an environment variable use to match any imagination...

As a security engineer, offensive or defensive, knowing how to find what is exposed and being able to access these information is almost necessary skill to have.

And lets not kid our selfs, this is definetely a skill you gonna appreciate while playing on our online platform echoCTF.RED, since environment variables seem harder to get than root user access...

## How they work
We'll use a variety of systems in order to examine how the environment variables are behaving. When a system or shell is not mentioned you can safely assume its bash.

Lets get started with defining a few variable and using them
```bash
$ myint=1
$ mystring="This is a long string"
$ echo $myint
1
$ echo $mystring
This is a long string
```

If from the same terminal you run bash again you'll notice the variables have disappeared.
```bash
$ echo $mystring
This is a long string

$ bash
$ echo $mystring

$
```

The same will happen if you open another terminal and try to access the variable. So why is that?
The variables have effect only on the current session. In order for a variable to be available on subsequent commands and sessions that are spawned from the existing one, we have to `export` them.
```bash
$ export mystring
$ bash
$ echo $mystring
This is a long string
```

Most shells and many commands use configuration files when they start (eg `~/.bashrc`) to instruct them on setting variables. Furthermore, certain commands set their own variables in order to help subsequent commands (eg `USER` being set by the login etc).

### docker variables
Understanding the concept of variable visibility is particularly important in situations where environment variables are set from the system boot (such as docker containers), which under certain conditions, makes them being exported to the global process space.

_We will not go deep into docker specifics just as far as we need for understanding the variables._

Lets see an example with docker variables.
```bash
$ docker run -it -e "myvar=myvalue" bash
root@envlab: / # echo $myvar
myvalue
root@envlab: / #
```

One thing we notice is that this variable is exported
```bash
root@envlab: / # bash
root@envlab: / # echo $myvar
myvalue
```

Even if we start a login bash the variable is still there
```bash
root@envlab: / # bash -l
4df007c4e761:/# echo $myvar
myvalue
4df007c4e761:/#
```

Even if we change users and shell, the variable is still there
```bash
4df007c4e761:/# su -s /bin/sh bin
4df007c4e761:/$ id
uid=1(bin) gid=1(bin) groups=1(bin),1(bin),2(daemon),3(sys)
4df007c4e761:/$ echo $myvar
myvalue
```

However, as we can understand this could have some very dangerous side effects, for this reason, many daemons running on your system choose to clear the environment for subsequent processes (eg php-fpm which has `clear_env = yes` by default). This is the reason you cannot access the `ETSCTF_FLAG` when you get shell as `www-data`.

## How to get them
So lets go to the juicy part on how to find and display the values of these variables.

### `printenv` - print all or part of environment
The `printenv` can be used to see the current variables defined in the _environment_
```bash
root@envlab: / # printenv
HOSTNAME=envlab
PWD=/
HOME=/root
_BASH_VERSION=5.1.8
_BASH_BASELINE=5.1
_BASH_LATEST_PATCH=8
TERM=xterm
SHLVL=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/bin/printenv
root@envlab: / #
```

### `env` - print the environment or run a program in a modified environment
The `env` and `printenv` commands behave exactly the same when used without parameters. However, `env` can also be used to run a command with modified environment.

1. print the current environment
```bash
root@envlab: / # env
HOSTNAME=envlab
PWD=/
HOME=/root
_BASH_VERSION=5.1.8
_BASH_BASELINE=5.1
_BASH_LATEST_PATCH=8
TERM=xterm
SHLVL=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/env
root@envlab: / #
```

2. Reset the environment and run `/usr/bin/env` to print the environment and see that its empty
```bash
root@envlab: / # env -i /usr/bin/env
root@envlab: / #
```

3. Reset the environment and run `/usr/local/bin/bash`. What we get is the variables that bash defines by default
```bash
root@envlab: / # env -i /usr/local/bin/bash
root@envlab: / # env
PWD=/
SHLVL=1
_=/usr/bin/env
root@envlab: / #
```

4. Run bash with modified `$HOME` variable
```bash
root@envlab: / # env -i HOME=/home /usr/local/bin/bash
root@envlab: / # echo $HOME
/home
root@envlab: / # env
PWD=/
HOME=/home
SHLVL=1
_=/usr/bin/env
```

### `set` BUILTIN - show or set shell variables and functions
Without options, the name and value of each shell variable are displayed in a format that can be reused as input for setting or resetting the currently-set variables. Read-only variables cannot be reset.

This is a builtin command that is used to define and/or print shell specific variables and functions.

Running just `set`, returns far more variables than before.
```bash
root@envlab: / # set
BASH=/usr/local/bin/bash
BASHOPTS=checkwinsize:cmdhist:complete_fullquote:expand_aliases:extquote:force_fignore:globasciiranges:hostcomplete:interactive_comments:progcomp:promptvars:sourcepath
BASH_ALIASES=()
BASH_ARGC=([0]="0")
BASH_ARGV=()
BASH_CMDS=()
BASH_LINENO=()
BASH_SOURCE=()
BASH_VERSINFO=([0]="5" [1]="1" [2]="8" [3]="1" [4]="release" [5]="x86_64-pc-linux-musl")
BASH_VERSION='5.1.8(1)-release'
COLUMNS=211
DIRSTACK=()
EUID=0
GROUPS=()
HISTFILE=/root/.bash_history
HISTFILESIZE=500
HISTSIZE=500
HOME=/root
HOSTNAME=envlab
HOSTTYPE=x86_64
IFS=$' \t\n'
LINES=55
MACHTYPE=x86_64-pc-linux-musl
MAILCHECK=60
OPTERR=1
OPTIND=1
OSTYPE=linux-musl
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PPID=0
PS1='\s-\v\$ '
PS2='> '
PS4='+ '
PWD=/
SHELL=/bin/ash
SHELLOPTS=braceexpand:emacs:hashall:histexpand:history:interactive-comments:monitor
SHLVL=1
TERM=xterm
UID=0
_=bash
_BASH_BASELINE=5.1
_BASH_LATEST_PATCH=8
_BASH_VERSION=5.1.8
```

These are variables that are defined by our current shell instance and unless otherwise tweaked, will not be inherited by child processes. For instance we can see that the `BASH_` related variables are not set on the invocation of the followup shell (`ash` in our example)
```bash
root@envlab: / # /bin/ash
/ # set
HISTFILE='/root/.ash_history'
HOME='/root'
HOSTNAME='envlab'
IFS='
'
LINENO=''
OPTIND='1'
PATH='/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
PPID='1'
PS1='\w \$ '
PS2='> '
PS4='+ '
PWD='/'
SHLVL='2'
TERM='xterm'
_='/bin/sh'
_BASH_BASELINE='5.1'
_BASH_LATEST_PATCH='8'
_BASH_VERSION='5.1.8'
/ #
```

An added bonus of using `set` is that it also displays functions that may have been defined
```bash
root@envlab: / # function mytest() { ls; }
root@envlab: / # set|tail
TERM=xterm
UID=0
_=set
_BASH_BASELINE=5.1
_BASH_LATEST_PATCH=8
_BASH_VERSION=5.1.8
mytest ()
{
 ls
}
root@envlab: / # mytest
bin dev etc home lib media mnt opt proc root run sbin srv sys tmp usr var
```
### `declare` / `typeset` BUILTINS - declare variables and/or give them attributes
Declare variables and/or give them attributes. If no names are given then display the values of variables. This command has the ability to display and manipulate variables as well as their attributes (eg. int, array, exported, etc).

Without arguments it operates just like `set`, displaying the shell and environment variables and functions
```bash
envlab:/# declare
BASH=/usr/local/bin/bash
BASH_ALIASES=()
BASH_ARGC=([0]="0")
BASH_ARGV=()
BASH_CMDS=()
BASH_LINENO=()
BASH_SOURCE=()
.
.
.
```

However if we run `declare -p` we can also see their attributes
```bash
envlab:/# declare -p
declare -- BASH="/usr/local/bin/bash"
declare -i BASHPID
declare -A BASH_ALIASES=()
declare -a BASH_ARGC=([0]="0")
declare -x PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
declare -ir UID="0"
```

This is what the declare attributes mean
* `-a` Each name is an indexed array variable
* `-A` Each name is an associative array variable
* `-f` Use function names only
* `-i` The variable is treated as an integer
* `-l` When the variable is assigned a value, all upper-case characters are converted to lower-case
* `-n` Give each name the nameref attribute, making it a name reference to another variable
* `-r` Make names readonly
* `-t` Give each name the trace attribute
* `-u` When the variable is assigned a value, all lower-case characters are converted to upper-case
* `-x` Mark names for export to subsequent commands via the environment

For more details about the `declare` command check the `bash(1)` manual pages under the **SHELL BUILTIN COMMANDS** section.

The `typeset` command behaves very similar to `declare` and is mostly there for compatibility with other shells. However, `typeset` without options, returns all variables and builtin shell functions.

### ps
The previous commands we saw all work on the current shell session, however many times we would like to see what environment variables were defined for an already running application, that may or may not have started by us.

The `ps` command, can help with that. This command provides information about running processes on a system, such as PID, name, running time, memory used etc. By adding a few extra options we can see a lot more information than just these. The options that are of interest to us, along with their meanings can be found below
* `a`: Select all processes except both session leaders.
* `e`: Show the environment after the command.
* `w`: Wide output. Use this option twice for unlimited width.
* `f`: ASCII art process hierarchy (forest).
* `u`: Display user-oriented format.
```bash
root@105b97a5ef81:/# ps -afeww
UID PID PPID C STIME TTY TIME CMD
root 1 0 0 08:48 pts/0 00:00:00 /bin/bash /entrypoint.sh bash
root 17 1 0 08:48 pts/0 00:00:00 bash
root 93 17 0 08:55 pts/0 00:00:00 ps -feww

root@105b97a5ef81:/# ps -aufeww
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
root 1 0.0 0.0 17964 2900 pts/0 Ss 08:48 0:00 /bin/bash /entrypoint.sh bash PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin HOSTNAME=envlab TERM=xterm DEBIAN_FRONTEND=noninteractive HOME=/root
root 17 0.0 0.0 18180 3348 pts/0 S 08:48 0:00 bash HOSTNAME=envlab PWD=/ HOME=/root DEBIAN_FRONTEND=noninteractive TERM=xterm SHLVL=1 PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin _=/bin/bash
root 94 0.0 0.0 36632 2908 pts/0 R+ 08:55 0:00 \_ ps -ufeww HOSTNAME=envlab PWD=/ HOME=/root DEBIAN_FRONTEND=noninteractive TERM=xterm SHLVL=2 PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin _=/bin/ps
```

We can see in the output above, that we get the environment variables as they exist for each of the running commands. However there are a few limitations. If we run the same commands as non root user we can see that we dont get to see the environment of processes from other users.
```bash
databus@63f7264d374e:/$ ps -aueww
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
root 1 0.0 0.0 17964 2992 pts/0 Ss 09:04 0:00 /bin/bash /entrypoint.sh bash
root 18 0.0 0.0 18184 3280 pts/0 S 09:04 0:00 bash
root 67 0.0 0.0 46844 2772 pts/0 S 09:09 0:00 su - databus
databus 68 0.0 0.0 4276 708 pts/0 S 09:09 0:00 -su TERM=xterm PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games MAIL=/var/mail/databus HOME=/ SHELL=/bin/sh USER=databus LOGNAME=databus
databus 71 0.0 0.0 18188 3292 pts/0 S 09:09 0:00 bash -l MAIL=/var/mail/databus USER=databus HOME=/ LOGNAME=databus TERM=xterm PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games SHELL=/bin/sh PWD=/
sampleu+ 152 0.8 0.0 18196 3056 pts/1 Ss+ 09:13 0:00 bash
databus 160 0.0 0.0 36632 2888 pts/0 R+ 09:14 0:00 ps -aueww USER=databus PWD=/ HOME=/ MAIL=/var/mail/databus SHELL=/bin/sh TERM=xterm SHLVL=1 LOGNAME=databus PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games _=/bin/ps
```

As we can see, the first three processes belonging to the `root` and `sampleuser` (one line before last) have no environment information included into the `ps` output.

### /proc
There is another way to get the environment from a running process and this is through `/proc`, the process information pseudo-filesystem.

The proc filesystem is a pseudo-filesystem which provides an interface to kernel data structures. It is commonly mounted at `/proc`, automatically by the system.

The filesystem provides and easy query to query and manipulate kernel structures as if they were simple files. For every process on the system, there is a corresponding directory under `/proc/<pid>/` with the exported kernel information. The files located under that folder correspond to different types of kernel information, but the ones that is of interest to us is `environ` & `cmdline`

`/proc/[pid]/environ`: This file contains the **initial** environment that was set when the currently executing program was started via `execve(2).` The entries are separated by null bytes (`\0`), and there may be a null byte at the end. Thus, to print out the environment of process 1, you would do
```bash
root@63f7264d374e:/# strings /proc/1/environ
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=envlab
TERM=xterm
DEBIAN_FRONTEND=noninteractive
HOME=/root
```

`/proc/[pid]/cmdline`: This read-only file holds the complete command line for the process, unless the process is a zombie. In the latter case, there is nothing in this file: that is, a read on this file will return 0 characters. The command-line arguments appear in this file as a set of strings separated by null bytes (`\0`), with a further null byte after the last string.
```bash
root@63f7264d374e:/# strings /proc/1/cmdline
/bin/bash
/entrypoint.sh
bash
```

The reason we want to examine the `environ` is quite clear, however, `cmdline` may not be. The reason we examine both files is that the command that got initiated may include environment variables on its command line.

Keep in mind, that depending on the mount option `hidepid=n` the `/proc` filesystem may behave as differently
* `0`: Everybody may access all `/proc/[pid]` directories. This is the traditional behavior, and the default if this mount option is not specified.
* `1`: Users may not access files and sub-directories inside any `/proc/[pid]` directories but their own (the `/proc/[pid]` directories themselves remain visible). Sensitive files such as `/proc/[pid]/cmdline` and `/proc/[pid]/status` are now protected against other users. This makes it impossible to learn whether any user is running a specific program (so long as the program doesn't otherwise reveal itself by its behavior).
* `2`: As for mode 1, but in addition the `/proc/[pid]` directories belonging to other users become invisible. This means that `/proc/[pid]` entries can no longer be used to discover the PIDs on the system. This doesn't hide the fact that a process with a specific PID value exists (it can be learned by other means, for example, by `kill -0 $PID`), but it hides a process's UID and GID, which could otherwise be learned by employing `stat(2)` on a `/proc/[pid]` directory. This greatly complicates an attacker's task of gathering information about running processes (e.g., discovering whether some daemon is running with elevated privileges, whether another user is running some sensitive program, whether other users are running any program at all, and so on).

## What next
I hope you enjoyed the reading and you were able to learn a few extra tricks in finding with environment variables for your session as well as running processes. All you have to do now is go back to echoCTF.RED and see if you can grab the environment variables from those targets that you still haven't finished 😂

Keep in mind that other shells (ash, ksh, csh etc) may behave differently regarding environment and builtin variables and functions. The commands outlined should be the same in most, however the output may differ. It is always a good idea to check the manual pages of the respective shell you're working with before trying random things.

## References
* <https://docs.oracle.com/cd/E19683-01/817-6958/userconcept-26/index.html>
* <https://www.digitalocean.com/community/tutorials/how-to-read-and-set-environmental-and-shell-variables-on-linux>
* <https://www.cs.ait.ac.th/~on/O/oreilly/unix/upt/ch06_01.htm>