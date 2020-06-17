Vim Tmux Navigator
==================

A way to navigate seamlessly between vim splits and tmux panes. Now portable and without bloat!

Christoomeys version of this script uses non portable options with the ps command, is generally clunky and is hard to read. The source gist by mislav, however, has a much more sensible way of checking whether vim is open or not.

It uses the following code
```
cmd="$(tmux display -p '#{pane_current_command}')"
cmd="$(basename "$cmd" | tr A-Z a-z)"

if [ "${cmd%m}" = "vi" ]; then
  direction="$(echo "${1#-}" | tr 'lLDUR' '\\hjkl')"
  # forward the keystroke to Vim
  tmux send-keys "C-$direction"
else
  tmux select-pane "$@"
fi
```

However, having a separate script is messy, so instead I used his method of getting the current panes as a sane implementation of `is_vim` in christoomeys version.

Christoomeeys implementation of `is_vim`:
```sh
ps -o state= -o comm= -t '#{pane_tty}' \
    | grep -iqE '^[^TXZ ]+ +(\\S+\\/)?g?(view|n?vim?x?)(diff)?$'
```
My implementation of `is_vim`:
```sh
tmux display -p | grep -q '[0-9]*:vi.'
```

I've also cut some of the """features""" from the vim plugin. If you wish to remap your keys, simply bind something to `:TmuxNavigateUp` `:TmuxNavigateDown` etc.

The backslash to go to latest window is also gone because I don't like it. If you want it, add it back in yourself. I believe in you!

A last change I've made is change the scripts from using bash to sh... The fact that people force you into using bash for no good reason...

Usage
-----

This plugin provides the following mappings which allow you to move between
Vim panes and tmux splits seamlessly.

- `<ctrl-h>` => Left
- `<ctrl-j>` => Down
- `<ctrl-k>` => Up
- `<ctrl-l>` => Right

Installation
------------

### Vim

If you don't have a preferred installation method, I recommend using [Vundle][].
Assuming you have Vundle installed and configured, the following steps will
install the plugin:

Add the following line to your `~/.vimrc` file

``` vim
Plugin 'depsterr/vim-tmux-navigator'
```

Then run

```
:source %
:PluginInstall
```

If you are using Vim 8+, you don't need any plugin manager. Simply clone this repository inside `~/.vim/pack/plugin/start/` directory and restart Vim.

```
git clone git@github.com:christoomey/vim-tmux-navigator.git ~/.vim/pack/plugin/start/
```


### tmux

To configure the tmux side of this customization there are two options:

#### Add a snippet

Add the following to your `~/.tmux.conf` file:

``` tmux
# Smart pane switching with awareness of Vim splits.
# See: https://github.com/depsterr/vim-tmux-navigator
is_vim="tmux display -p | grep -q '[0-9]*:vi.'"
bind-key -n 'C-h' if-shell "$is_vim" 'send-keys C-h'  'select-pane -L'
bind-key -n 'C-j' if-shell "$is_vim" 'send-keys C-j'  'select-pane -D'
bind-key -n 'C-k' if-shell "$is_vim" 'send-keys C-k'  'select-pane -U'
bind-key -n 'C-l' if-shell "$is_vim" 'send-keys C-l'  'select-pane -R'

bind-key -T copy-mode-vi 'C-h' select-pane -L
bind-key -T copy-mode-vi 'C-j' select-pane -D
bind-key -T copy-mode-vi 'C-k' select-pane -U
bind-key -T copy-mode-vi 'C-l' select-pane -R
```

#### TPM

If you'd prefer, you can use the Tmux Plugin Manager ([TPM][]) instead of
copying the snippet.
When using TPM, add the following lines to your ~/.tmux.conf:

``` tmux
set -g @plugin 'depsterr/vim-tmux-navigator'
run '~/.tmux/plugins/tpm/tpm'
```

Why don't you support X?
------------------------

Cause I don't use it. Edit the source or go make a fork.
