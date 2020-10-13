---
title: "Going Full Bananacakes"
date: "Mon, Mar 18, 2019"
layout: post
tags: linux tmux ssh
cover: img/banana/1.png
---

Spring Break occurred last week, and I did a few things to make my life easier, with satisfactory results. 

$index{min_depth=2}

## Introduction
I had a showerthought:
> "What if I just make a server at home with all my s***t on it and log into it to do everything? I generally know in advance if I won't have internet access, and I mainly just need a consistent dev environment..."

So I did some searching, and yes, others have had the same want, _and_ they've gone and done the thing:

- [https://jann.is/ipad-pro-for-programming/](https://jann.is/ipad-pro-for-programming/)
- [https://bergie.iki.fi/blog/six-weeks-working-android/](https://bergie.iki.fi/blog/six-weeks-working-android/)

## Hardware
I got the cheapest machine I could find which would run g++-8 at tolerable speed, among my other tools. 
Specifically, I got [one of these](https://amzn.to/2FnS1sJ). 

Docker is probably a no-go on the Intel Atom micro pc I bought, but that's just fine for now. Due to modem placement and wanting ethernet speeds, my server has to be located in my roommate's bedroom, so its fanless, tiny design is a good thing, as I'm not taking up loads of her space or making noise.

## One Environment
The micro pc came with Windows 10 installed--that had to go. I slapped Ubuntu on there and installed my typical tools and dependencies. A daily cron job backs up my files to an external HDD, current projects are usually in Dropbox, and `scp` covers the gaps.

## The Zen of Tmux
I had never used `screen` or `tmux` until last week. I've used i3, I've manually tiled windows, I've used tabs and panes in terminal emulators and text editors supporting them, but... Somehow I had never seen or tried `tmux`. And holy butts, is it dope.

### Sensibility
The default keybindings for tmux don't make much sense coming from a modern computing experience. Presumably, they made more sense when tmux was originally developed. In any case, the first order of business was to `man tmux` and go write a `~/.tmux.conf` to set sensible key bindings.

```bash
# more sensible splits
bind | split-window -h
bind - split-window -v
unbind '"'
unbind %
# reload config conveniently
bind r source-file ~/.tmux.conf
# move between panes
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D
# mouse support
set -g mouse on
```

### Persistence
The next order of business was to make sure I never have to set up my panes again. To do so, I needed to install [Tmux Plugin Manager](https://github.com/tmux-plugins/tpm), then install [Tmux Resurrect](https://github.com/tmux-plugins/tmux-resurrect), _then_ install [Tmux Continuum](https://github.com/tmux-plugins/tmux-continuum). With all that done and tested, my tmux sessions now persist across reboots.

### Nested Sessions
`~/tmux.conf` varies slightly between the server and my laptop, as _nested_ tmux sessions can be a bit hinky: if you have redundant key bindings between them, the outer session will consume keystrokes you intended for the inner one.

I found a [clever way](http://stahlke.org/dan/tmux-nested/) around this, which is more complex than I needed. Prefixed bindings are easy: `ctrl+b d` becomes `ctrl+b b d` to reach the inner tmux session. For moving among panes, I added a _second_ set of bindings to the server's config, allowing for `shift+arrow` in addition to `alt+arrow`. This way, if the server session is nested, I can still get around without issue, and if it _isn't_ nested, I can use the bindings I'm used to.

## Nano Ain't so Bad
No, I still haven't gotten comfy in emacs or (neo)vim. Of the two, I will probably learn vim ultimately. In the meantime, I enjoy my graphical editors but can tolerate nano. With custom (modernized) keybindings and the addition of linting in the new versions, it is a perfectly tolerable editor. I still need to write a syntax file for [MoonScript](https://moonscript.org), but nano is mostly quite usable these days.

## Accessing the Server
Ok, so I have a server running Ubuntu with all my dev tools and a cozy cli. Next: set up an ssh server with a custom port, set up mosh, ignore all other incoming network traffic, and disable ping. Then, I went ahead and copied a key over from my lappy, so I don't have to type passwords constantly. Then, I set up appropriate port forwarding on my router, so I can log into my server from wherever.

## Results: Remote Work Anytime, Anyplace, on Anything
So, with all that, I can work exactly the same on my ancient Macbook Air, my workable ASUS Zenbook, or a Chromebook, or Android with [JuiceSSH](https://telegram.org), or iOS with [Blink](https://www.blink.sh), with much better battery life than running builds and stuff locally.

### Going Further Bananacakes
Shortly after setting all this stuff up and moving my general workflow over, I had a good opportunity to try something unfamiliar. I've taken to using [todo.txt](http://todotxt.org) lately. I also recently weighed myself for the first time in several months. After realizing that I somehow gained a whopping 40lb last year (curse you, alfredo), I wrote a little python script to generate todo entries prescribing calorie, hydration, and exercise goals for the day. Then, I made a [Telegram](https://telegram.org) bot, and wrote _another_ python script to send myself bot messages. Then I made a cron job on my server that, each morning, gets a list of my top-priority todos, prettifies them a little, and sends them to my phone via Telegram. I didn't really anticipate the convieniences of running a little server.

## What's Next
If this setup remains as deliciously convenient as it feels so far, I will eventually need to upgrade it a bit: a vpn maybe, a beefier server machine, etc. I really like the idea of using an iPad Pro 12.9" as my "client" machine. The iPad was already pretty nice for doing art, writing, or music stuff, but now I could do that stuff _and_ code on it, too? Oof. Maybe in a year or two. ;)
