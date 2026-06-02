# tmux Cheatsheet
### Vi/Vim mode:
```
set-window-option -g mode-keys vi
```
### Clipboard w/ vi mode (`xclip`)
```
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel 'xclip -in -selection clipboard'
```
### `.tmux.conf`
Sets vi mode, if plugins are enabled, does logging, takes care of `xclip` clipboard in vi mode, sets bindings for changing pane name:
```
set-option -g default-shell /bin/zsh
set -g @plugin 'tmux-plugins/tmux-logging'
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g history-limit 250000
set -g allow-rename off
set -g escape-time 50
set-window-option -g mode-keys vi
run '/root/.tmux/plugins/tpm/tpm'
run '/root/.tmux/plugins/tmux-logging/logging.tmux'
run '/root/.tmux/plugins/tmux-logging/scripts/toggle_logging.sh'
bind-key "c" new-window \; run-shell "/root/.tmux/plugins/tmux-logging/scripts/toggle_logging.sh"
bind-key '"' split-window \; run-shell "/root/.tmux/plugins/tmux-logging/scripts/toggle_logging.sh"
bind-key "%" split-window -h \; run-shell "/root/.tmux/plugins/tmux-logging/scripts/toggle_logging.sh"

# Custom
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel 'xclip -in -selection clipboard'
set -g pane-border-status top
set -g pane-border-format "#{pane_index} #{pane_title}"
# rename prompt
bind . command-prompt -p "(rename-pane)" -I "#T" "select-pane -T '%%'"
```


> [!Resources]
> - [Tmux Cheatsheet](https://tmux.info/docs/cheatsheet)
> - [My .tmux.conf](https://github.com/PentestPuppy/conf/blob/main/.tmux.conf)
