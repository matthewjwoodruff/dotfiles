# Dotfiles

2018-01-11 12:22

I generally dislike configuration, and I think one ought
to be able to sit down at any system and use the standard
tools.

That being said, there are a few configuration options
I find useful, without which I find myself plagued by
annoyance.

It's important to understand these options thoroughly
before applying them, however.  To that end, this dotfiles
repo contains only notes about what to put in your config
files, rather than providing plug-and-play configurations.

# Tmux

## High Priority

I used `screen` long before I discovered `tmux`.  As a
result, ^B fights against my muscle memory:

```
unbind C-b
set -g prefix C-a
bind C-a send-prefix
```

Although I like the vi keybindings for text editors,
I mostly prefer to use vanilla keybindings elsewhere.
But I really like them for copy-mode in tmux:

```
set-window-option -g mode-keys vi
```

## Low Priority

These make navigating and resizing panes more pleasant.
Unfortunately, 5 characters at a time is a necessary
compromise between big and little screens.

```
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

bind < resize-pane -L 5
bind > resize-pane -R 5
bind - resize-pane -D 5
bind + resize-pane -U 5
```

Finally, I'm almost always using a 256-color terminal, and
if I'm not, I know what that looks like and how to fix it:

```
set -g default-terminal screen-256color
```

# i3

## High Priority

Every system has different terminal emulators that work
well on it.  Often I end up in the `urxvt` manpage trying
to customize the terminal.  Here are some more-or-less
sensible terminals:

```
bindsym $mod+Shift+Return exec "urxvt -bg gray70 -rv -fade 5 +sb -vb -tn xterm-256color -fn 'xft:Source Code Pro Medium:pixelsize=24' "
bindsym $mod+Return exec "gnome-terminal --profile=smaller"
```


The biggest advantage to using gnome-terminal over urxvt is
that it's tuned in to whatever foolishness other GUI apps
are using for copypaste instead of left-click selection
and middle-click paste.  (I wish web browsers would honor
that convention consistently!)

Navigation keybindings default to the uncanny valley:
`jkl;` instead of `hjkl`.  Just close enough to be
maddenning.

You can define them at the top and then use them later:

```
set $up h
set $down j
set $left k
set $right l
```

But if you do this, you have to change how splits are
created.  I like to add Ctrl:

```
# split in horizontal orientation
bindsym Mod1+Ctrl+h split h

# split in vertical orientation
bindsym Mod+Ctrl1+v split v
```

## Low Priority

I tend not to hit mod+E by mistake, so I prefer to exit
directly and skip the nagbar:

```
bindsym $mod+Shift+e exec "i3-msg exit"
```

# vim

## Low Priority

I mostly use Kakoune now, but I may not have bothered
to build it, or I may need it for one of the things it's
better at.  (Interactive `sed` :), and vimdiff)

I like to turn on syntax highlighting and autoindent,
highlight matches, and disable the mouse.

```
filetype plugin on
filetype plugin indent on
syntax on
set hls
set mouse=
set number
```

# Kakoune

Three cheers for kak!  It's a delightful text editor.

## High Priority

I tend to use spaces for indent because I spend so much
time writing Python.  And for Python, 4-space tabs are
not optional.  I don't have deep personal feelings one
way or the other.  Just try to be consistent, people:

```
hook global WinCreate .* %{
    hook window InsertChar \t %{ exec -draft -itersel h@ }
}
map global insert <backtab> '<a-;><lt>'
set-option global tabstop 4
```

Honestly kak configuration syntax isn't something I've
devoted a lot of effort to learning.  See my comments
above about preferring the defaults.  Interestingly,
vim has a whole lot more options, so each config line is
shorter, but kak's commands are simple and compose well.
You need to write more stuff as a result to get the same
effect, but kak lets you have a slightly different effect
if you prefer.

Anyhow, if you want real tabs for Makefiles, you can also
do:

```
map global insert <a-t> '<a-;>!printf "\t"<ret>'
```

You could map from `<a-tab>` instead, but that messes with
the muscle memory from Windows, so I wouldn't recommend it.
It's bad enough that I'm used to using mod+L to switch
windows in i3, which locks the screen on MS Windows.

## Low Priority

The mouse drives me absolutely bonkers:

```
set-option global ui_options ncurses_enable_mouse=false
```


Number lines and show matches:

```
hook global WinCreate .* %{add-highlighter buffer number_lines}
hook global WinCreate .* %{add-highlighter buffer dynregex '%reg{/}' 0:MenuForeground}
```

These two shortcuts for timestamp and text wrapping are
of my own devising, and I use them very heavily:

```
map global insert <c-t> %{<a-;>!date +"%Y-%m-%d %H:%M"<ret>}
map global normal <c-w> %{<a-x>|fmt -60<ret>l}
```

# Copyright Statement: CC0

This document (`README.md`) is licensed
[CC0](https://creativecommons.org/publicdomain/zero/1.0/legalcode).