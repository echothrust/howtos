# Using Xnest to play with window managers
<sub>This post was originaly posted some 9 or 10 years ago</sub>

Often times we want to test different things without messing up our existing
X11 sessions. Such tests are the window manager ones. You want to tweak your
manager to your liking but you don't want to restart your current session.
Xnest is the tool that you need to know. Xnest allows one to connect to
existing xservers and emulating a client connection. In laments terms you can
run a window manager inside a window.


# Configuring xinit
Just like you would normally do, we have to make our xinit to act to our
likings. Setting up backgrounds, choosing window managers etc. However since
you might already have a local .xinitrc file we will create a new one. Here is
how our looks like

```sh
------~/.xinitrc.nest-------
#!/bin/sh
# $Xorg: xinitrc.cpp,v 1.3 2000/08/17 19:54:30 cpqbld Exp $
# $OpenBSD: xinitrc.cpp,v 1.13 2006/01/21 22:12:13 matthieu Exp $
userresources=$HOME/.Xresources
usermodmap=$HOME/.Xmodmap
sysresources=/usr/X11R6/lib/X11/xinit/.Xresources
sysmodmap=/usr/X11R6/lib/X11/xinit/.Xmodmap
MCOOKIE=deadbeef

display=$(ls /tmp/.X*-lock|wc -l|awk '{print $1}')
DISPLAY=:$display.0
xauth add $(hostname)/unix${DISPLAY} . $MCOOKIE
xauth add localhost/unix${DISPLAY} . $MCOOKIE

export DISPLAY
xsetroot -cursor_name left_ptr
feh -F -Z --bg-scale  ~/.fvwm/echothrust.jpg  
xterm -ls -name XTBIG &
# merge in defaults and keymaps

if [ -f $sysresources ]; then
    /usr/X11R6/bin/xrdb -merge $sysresources
fi

if [ -f $sysmodmap ]; then
    /usr/X11R6/bin/xmodmap $sysmodmap
fi

if [ -f $userresources ]; then
    /usr/X11R6/bin/xrdb -merge $userresources
fi

if [ -f $usermodmap ]; then
    /usr/X11R6/bin/xmodmap $usermodmap
fi

for fontpath in `grep FontPath /etc/X11/xorg.conf |awk '{print $2}'|sed -e 's/"//g'`;
do
	xset +fp ${fontpath}
done
	xset fp rehash
feh -F -Z --bg-scale  ~/.fvwm/echothrust.jpg  
exec fvwm  -display $DISPLAY
xauth remove $(hostname)/unix${DISPLAY} localhost/unix${DISPLAY}

------~/.xinitrc.nest-------
```

Tune this to your liking.


# Starting Xnest
Now to nasty part is to start the nested X. Change this to the desired display.
```sh
xinit .xinitrc.nest -- /usr/X11R6/bin/Xnest -full -dept 24 :2
```

{{tag>howto XOrg OpenBSD Xnest}}
