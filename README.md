# Dotfiles

2019-09-07 09:53

This file lists the options I like for some tools that
I use.

There's a balance between spending too little time on
configuration and spending too much, between using tools
that don't fit the hand and being the artist who sharpens
pencils for five hours without drawing anything.  I err
on the side of under-configuring things.

# Tmux

## High Priority

I like `^A` rather than `^B` because I switched from
`screen` and got used to `^A`.

```
unbind C-b
set -g prefix C-a
bind C-a send-prefix
```

I mostly avoid `vi`-style keybindings outside of text
editors, but I like them for the `tmux` copy mode too.

```
set-window-option -g mode-keys vi
```

## Low Priority

These make navigating and resizing panes easier.
5 characters at a time is a compromise between big and
little screens.

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

### TrueColor

This doesn't exactly have "robust solution" written all
over it, but it gives me pretty colorschemes in Kakoune.

```
set -g default-terminal tmux-256color
set -ga terminal-overrides ",xterm-256color:Tc"
```

I think I stole this from the Arch Wiki.

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
and middle-click paste.  (I wish web browsers would use
that convention!)

Lately I've been using kitty a lot too.

`i3` navigation keybindings default to the uncanny valley:
`jkl;` instead of `hjkl`.  It's just close enough to
be maddenning.

You can define them at the top and then use them later,
for move, rearrange, and resize (3 places!):

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

I prefer to exit directly and skip the nagbar:

```
bindsym $mod+Shift+e exec "i3-msg exit"
```

Usually the important stuff is in a Kakoune session I
started with `kak -s`, so even if I kill my session by
mistake, it's easy to bounce back.

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

You could map to `<F1>` to `<Esc>` instead, but the
absolute best thing to do is to start using `^[` instead
of escape, because the `[` is more reliably located and
closer to the home row.

Laptop keyboards can be a problem here too, unfortunately,
because some of them have a `Fn` key to the left of
`Ctrl`, and others put it between `Ctrl` and the logo key.
You could remap `Ctrl` to Caps Lock, but I like Caps Lock
where it is.

# Kakoune

If you overuse visual mode in Vim and feel guilty about
it, consider using Kakoune.  Kakoune is modal like Vim,
but it's oriented around selections rather than actions.
I use it as my primary editor for writing programs.

## High Priority

I tend to use spaces for indent because I spend so much
time writing Python.  And for Python, 4-space tabs are
not optional.  I don't have deep personal feelings one
way or the other.  Just try to be consistent, people!

```
hook global WinCreate .* %{
    hook window InsertChar \t %{ exec -draft -itersel h@ }
}
map global insert <backtab> '<a-;><lt>'
set-option global tabstop 4
```

Honestly kak configuration syntax isn't something I've
devoted a lot of effort to learning.  Interestingly,
vim has a whole lot more options, so each config line is
shorter, but kak's commands are simple and compose well.
You need to write more stuff as a result to get the same
effect, but kak lets you have a slightly different effect
if you prefer.

Anyhow, if you want real tabs for Makefiles or something,
you can also do:

```
map global insert <a-t> '<a-;>!printf "\t"<ret>'
```

You could map from `<a-tab>` instead, but that messes with
the muscle memory from Windows, so I wouldn't recommend it.
It's bad enough that I'm used to using `mod4+L` to switch
windows in `i3` and `sway`, which locks the screen on
MS Windows.

## Low Priority

The mouse drives me bonkers.  If I want mouse select
at all, I want terminal-level mouse select, not
application-level mouse select.

```
set-option global ui_options ncurses_enable_mouse=false
```

I usually number lines and show matches.

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

# Sway

For keybindings, see i3 config.  I also use `swayidle`
for locking, similar to the commented lines in
`/etc/sway/config/`:

```
exec swayidle \
    timeout 600 'swaylock -c 007722' \
    timeout 1200 'swaymsg "output * dpms off"' \
       resume 'swaymsg "output * dpms on"' \
    before-sleep 'swaylock -c 007722'
```

# w3m

Text-mode browser means text-mode!  I prefer my internet
wordy and monochrome!

`alias w3m=w3m -M -no-mouse -o auto_image=FALSE`

I realize you can get this by building a minimal `w3m`
yourself, but I'm lazy.

# Copyright Statement: CC0

This document (`README.md`) is licensed
[CC0](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
