+++
title = 'Various macOS tools I use daily'
date = 2024-10-14T00:32:10-05:00
tags = ["macos", "utils"]
featured_image = '/posts/macos-utils/feature.png'
draft = false
+++

# Intro

A couple years back I decided to move off Windows for my daily driving. I was mainly tired of the poor performance and battery life of modern x86_64 laptops, and Microsoft's continued push to bloat up the OS with junk on even the most basic installs. For quite a few years I spent doing a lot of SCCM/MECM endpoint management for Windows desktop/servers, so this was a big change for me. After 2 years I can definitely say it's been a great journey. Before settling on macOS, I did run Arch briefly, but due to minor annoyances with software package support, minor issues with laptops, mainly around high DPI displays and iGPU + dGPU handling, I settled on macOS. These days I still have a beefy x86_64 Windows machine for running VMware Workstation to virtualize EVE-NG and Containerlab, but I spend the vast majority of my day in macOS. I currently use a Macbook Pro M1 Pro 14" at home, but also use a MacBook Air M2 for work.

A lot of the tools below have very good, if not better, Linux support as well. So if you're running Linux that means you might want to check them out too! And of course, WSL may be an option for many of these tools too.

# Software
## Neovim

I've used a lot of IDE's over the years, and I have to say neovim is my favorite thus far. When I was younger Notepad++ was my go to for anything, and I eventually moved to doom emacs after a friend showed me how utterly powerful it was, and briefly tried vscode. Neovim has really been great, partially due to Lua which I've had a decent amount of exposure over the years due to Gmod addon scripting. I personally use [LazyVim](http://www.lazyvim.org) as my distribution of choice. Although making your own nvim config from scratch can be worth it, I find that the defaults from LazyVim are good, and the only customization I have is using fzf over telescope, dashboard over alpha, and adding [wakatime](https://wakatime.com). For nerdfonts I use [Iosevka](https://github.com/be5invis/Iosevka). The theme in the picture below is Github Dark.

If you're looking for SecureCRT highlighting, feel free to check out the vim plugin [cisco.vim](https://github.com/momota/cisco.vim), or for Juniper JunOS- [junos.vim](https://github.com/momota/junos.vim). A way to run this in neovim is by launching neovim, opening a terminal session via `:term`, and then doing `:set ft=cisco` or `:set ft=junos`.

![](/posts/macos-utils/neovim.png)

## kitty

A network engineer tends to sit in CLI a lot, so having a good terminal is key. Like most, I started on PuTTY, maybe tried SecureCRT, but I can safely say there are much better tools out there. Up until recently I was an Alacritty + tmux user. I still use tmux here and there on remote hosts, but on my macOS I found that the persistent sessions were useless for my workflow. After reading Kovid Goyal's, the developer of kitty, thoughts on multiplexers [here](https://sw.kovidgoyal.net/kitty/faq/#i-am-using-tmux-and-have-a-problem), I embraced a more forward ideology. Since then, adapting to kitty has been bumpy, but worth it. My big use cases for kitty is using the native scrollback_pager functionality to pipe the last command output (kitty_mod+g by default), or the current scrollback (kitty_mod+h by default) to neovim. You can do this by adding the below line to your kitty.conf file:

```
scrollback_pager nvim -c "set signcolumn=no showtabline=0" -c "silent write! /tmp/kitty_scrollback_buffer | te cat /tmp/kitty_scrollback_buffer - "
```
Otherwise the use of tabs, windowing, layouts is all very similar to tmux. My config is pretty much the same as what's stock other than the above. Be sure to check out the [kittens available](https://sw.kovidgoyal.net/kitty/kittens_intro/). I am a big fan of using `kitten ssh` to ssh into devices so that when making a new pane in the same window, it automatically SSHs into the host I am currently SSH'd into.

## yazi

[Yazi](https://github.com/sxyazi/yazi) is a great TUI-based File Browser. It uses vim motions for navigating, has image preview along with typical file preview like text files. I'll admit I'm still getting used to using it on a daily basis but the more I work with yazi the more impressed I get with the flexibility.

![](/posts/macos-utils/yazi.png)

## Zoxide

[Zoxide](https://github.com/ajeetdsouza/zoxide) is a much more modern cd for those jumping around in terminal. The easiest thing to use zoxide for is doing something like `z github` to quickly jump to your github folder from any directory you are currently located in. It remembers your directories you use and provides suggestions, and using things like `zi github` can interactively select where you want to go. Just be aware you need fzf for that I believe.

## Shottr

I hope one day macOS's default screenshot utility can reach what [shottr](http://shottr.cc) does for me. Although shottr is missing some features I'd prefer to have such as automatic upload to an image hosting service, I like it more for easily preparing/prettifying screenshots for knowledge base articles. It can be a bit clunky at times when trying to merge multiple pictures, though this has gotten better in recent versions. Things like the pin feature, counter, and backdrop are my most used by far. I hope the developer can add the text box feature to not require a colored backdrop, but otherwise I very much like shottr. The screenshot below shows the counter feature. Great way to condense multiple screenshots into one when it works.

![](/posts/macos-utils/shottr.png)

## One last thing..

I won't cover it here, but I highly recommend you check out [this](https://www.josean.com/posts/7-amazing-cli-tools) blog post on some additional CLI tools I also use. Josean Martinez also has a youtube covering it, to show you how it all looks realtime. Using fzf + bat + delta + eza is very useful. I don't personally use tldr and tf, but I'm sure they're great!
