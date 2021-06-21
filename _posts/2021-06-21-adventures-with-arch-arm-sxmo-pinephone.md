---
layout: post
title:  "adventures with arch linux arm and sxmo on the pinephone"
categories: linux pinephone kernel archlinux arm sxmo
---

# WIP!

## Introduction
I just discovered SXMO for the Pinephone yesterday and im amazed by it, its excatly how i imagined myself using the Pinephone and even better. I decided to look deeper into it. First of all there is only a prebuilt image for SXMO using PostmarketOS, which is nice, but i really prefer Arch Linux ARM.

## first plan of attack
is to get a prebuilt SXMO image into the [Danctnix github repository](https://github.com/dreemurrs-embedded/Pine64-Arch). there is a [sxmo-alarm](https://github.com/justinesmithies/sxmo-alarm) with a nice script to install sxmo from the barebones image.

# build dependencies
{% highlight bash %}
sudo pacman -S build-essentials fakeroot bc flex bison autoconf
{% endhighlight %}

# configure
{% highlight bash %}
git clone https://github.com/dreemurrs-embedded/Pine64-Arch
cd Pine64-Arch/PKGBUILDS/pine64/linux-pine64/
vim config
makepkg -s
{% endhighlight %}


# Links

* [Github Repository Arch ARM Danctnix](https://github.com/dreemurrs-embedded/Pine64-Arch)
* [Arch ARM Danctnix Releases](https://github.com/dreemurrs-embedded/Pine64-Arch/releases)
* [Current Barebones Arch ARM](https://github.com/dreemurrs-embedded/Pine64-Arch/releases/download/20210613/archlinux-pinephone-barebone-20210613.img.xz)
* [sxmo repository](https://github.com/dreemurrs-embedded/Pine64-Arch)
* [sxmo-alarm repository](https://github.com/justinesmithies/sxmo-alarm)
* Matrix: #archmobile:kde.org
* IRC: #sxmo on irc.oftc.net
