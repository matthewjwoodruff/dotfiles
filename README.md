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
bind * rotate-window
```

Finally, I'm almost always using a 256-color terminal, and
if I'm not, I know what that looks like and how to fix it:

```
set -g default-terminal screen-256color
```

### Alternative TrueColor Fix

This has made my kak colorschemes work.  Instead of
`screen-256color` do these two lines:

```
set -g default-terminal tmux-256color
set -ga terminal-overrides ",xterm-256color:Tc"
```

I'm not sure where I got them, but they're in the Arch
wiki among other places.

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
to build it, or I may need vim for one of the things it's
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

Also, some keyboards put F1 where ESC belongs.  Especially
some versions of the Thinkpad keyboard.  If I want help,
I'll ask for it with `:help`.  I never ever want F1 to
mean help.

```
noremap <F1> <nop>
inoremap <F1> <nop>
```

You could map to <Esc> instead, but the absolute best thing
to do is to start using `^[` instead of escape, because the
`[` is more reliably located and closer to the home row.

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
hook global WinCreate .* %{add-highlighter buffer/ number-lines}
hook global WinCreate .* %{add-highlighter buffer/ dynregex '%reg{/}' 0:MenuForeground}
```

These two shortcuts for timestamp and text wrapping are
of my own devising, and I use them very heavily:

```
map global insert <c-t> %{<a-;>!date +"%Y-%m-%d %H:%M"<ret>}
map global normal <c-w> %{<a-x>|fmt -60<ret>l}
```

# ssh-agent

## High Priority

It's *incredibly* useful to have an ssh-agent running
as long as you are logged in.  10h is the timeout.
Historically I haven't used one but I think it's a good
idea.  I shamelessly copied this approach from (Mark
A. Hershberger](http://mah.everybody.org/docs/ssh), who
claims the approach originates with Joseph M. Reagle.

Put this in `.bash_profile`:

```
SSH_ENV="$HOME/.ssh/environment"

function start_agent {
    echo "Initializing new SSH agent..."
    /usr/bin/ssh-agent -t 10h | sed 's/^echo/#echo/' > "${SSH_ENV}"
    echo succeeded
    chmod 600 "${SSH_ENV}"
    . "${SSH_ENV}" > /dev/null
}

if [ -f "${SSH_ENV}" ]; then
    . "${SSH_ENV}" > /dev/null
    ps -ef | grep ${SSH_AGENT_PID} | grep ssh-agent$ > /dev/null || {
        start_agent;
    }
else
    start_agent;
fi
```

Archwiki has a user-systemd approach that didn't exist
the last time I looked.  Might be a superior approach,
but I don't always have systemd, e.g. on Cygwin.  It also
has a dead simple version of the approach I'm using,
but this version feels a little nicer.

# System Beep

System beep is annoying.  I prefer the flashing terminal.
[TLDP](http://tldp.org/HOWTO/Visual-Bell-8.html) recommends
the following:

* In `.inputrc`: `set prefer-visible-bell`
* In `.bashrc`: `set bell-style visible`

You have to do this on remote machines too or your ssh
sessions will beep at you.

# Mail User Agent

## git send-email

This was a real pain to figure out.

In `~/.gitconfig`

```
[sendemail]
    smtpEncryption = tls
    smtpServer = <smtp.domain.tld>
    smtpUser = <email address>
    smtpServerPort = 587
    smtpPass = "password"
    smtpAuth = "LOGIN"
    smtpServerOption = starttls
```

I also installed:

```
perl-io-socket-ssl
perl-authen-sasl
```

Also, your repo needs a configuration item:

```
[sendemail]
    to = mailinglist@domain.tld
```

## mutt

This follows the Arch Wiki.  I may not quite have
everything right, but it seems to do the job.

```
set imap_user=<email address>
set imap_pass="password"
set folder=imaps://imap.domain.tld:993
set spoolfile=+INBOX
set header_cache = ~/.cache/mutt
set message_cachedir = "~/.cache/mutt"

set realname="Real Name"
set from = <email address>
set use_from = yes
set smtp_url=smtps://$imap_user:$imap_pass@smtp.domain.tld:465
set ssl_force_tls = yes
set ssl_starttls = yes
set smtp_authenticators="login"

set record = "+Sent"
```

The last line there copies sent mail to the "Sent" folder.

## Comment 

Why `git send-email` needs to use port 587 and not 465,
for the same SMTP server, I have no idea.  Could be
misconfiguration at my end.  Someday I'll learn how
email works.

# Copyright Statement: CC0

This document (`README.md`) is licensed
[CC0](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
