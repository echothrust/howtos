# Greek Keyboard on OpenBSD 4.8
In order to configure the Greek layout we have to edit the `/etc/X11/xorg.conf`.

Under Section InputDevice specify the following options:
```
  Option "XkbModel" "pc105"
  Option  "XkbRules" "xorg"
  Option "XkbLayout" "us,el"
  Option "XkbOptions" "grp:alt_shift_toggle"
```
