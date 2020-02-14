# TMUX Fast guide for day1
Author: Dimitris Zoakos \


## Keystrokes

   * `Ctrl-b ?` List all keybindings
   * `Ctrl-b c` Create new window
   * `Ctrl-b d` Detach current client
   * `Ctrl-b l` Move to previously selected window
   * `Ctrl-b n` Move to the next window
   * `Ctrl-b p` Move to the previous window

Kill the current window
   * `Ctrl-b &`
   * `Ctrl-b ,` Rename the current window
   * `Ctrl-b %` Split Vertically the current window into two panes
   * `Ctrl-b "` Split Horizontally the current window into two panes
   * `Ctrl-b q` Show pane numbers (used to switch between panes)
   * `Ctrl-b o` Switch to the next pane
   * `Ctrl-b q` Shows the pane numbers where you can choose your working pane


To rerun the config file without restarting the tmux server:
   * `Ctrl-b : source-file ~/.tmux.conf`

## Usefull Commands
   * `tmux list-sessions` (display current sessions)
   * `tmux attach -t <session no>` (attach to running session)



## Tips & Tricks
  * Run rtorrent on startup ?
```
su - rtorrent -c ‘tmux new -n a /usr/local/bin/rtorrent \; detach’ 2>&1 > /dev/null
```

  * Synchronize typing on multiple panes
```
 ctrl+b :set-window-option synchronize-panes [on|off]
```
