---
layout: post
title:  "adventures with arch linux arm and sxmo on the pinephone"
---
# WIP!

#### Introduction
I just discovered SXMO for the Pinephone yesterday and im amazed by it, its excatly how i imagined myself using the Pinephone and even better. I decided to look deeper into it. First of all there is only a prebuilt image for SXMO using PostmarketOS, which is nice, but i really prefer Arch Linux ARM.

#### first plan of attack
is to get a prebuilt SXMO image into the [Danctnix github repository](https://github.com/dreemurrs-embedded/Pine64-Arch). there is a [sxmo-alarm](https://github.com/justinesmithies/sxmo-alarm) with a nice script to install sxmo from the barebones image, but im not really satisfied with this.

## creating the sxmo PKGBUILDs for Arch ARM
As seen here [sxmo build all](https://git.sr.ht/~mil/sxmo-build/tree/master/item/sxmo_build_all.sh) there couple of packages needed for sxmo. i'm starting the progress of creating PKGBUILD files for them.

#### build dependencies
{% highlight bash %}
sudo pacman -S build-essentials fakeroot bc flex bison autoconf
{% endhighlight %}

#### starting off with a simple package sxmo-st
this one was quite easy to do because sxmo developer proycon on irc sent me links to [alpine linux repository](https://pkgs.alpinelinux.org/packages?name=sxmo*&branch=edge) with all sxmo packages. they use very similar styled APKBUILD files for packaging as Arch, seen here in this sxmo-st [APKBUILD](https://gitlab.alpinelinux.org/alpine/aports/-/blob/master/community/sxmo-st/APKBUILD). with some help of the [APKBUILD reference](https://wiki.alpinelinux.org/wiki/APKBUILD_Reference), [PKGBUILD reference](https://wiki.archlinux.org/title/PKGBUILD) and [Creating Packages Arch](https://wiki.archlinux.org/title/Creating_packages) i figured everything out and the only things i had to change were formatting and a couple of package names in makedepends and depends. i removed the line ["sed -i '/tic/d' Makefile"i](https://gitlab.alpinelinux.org/alpine/aports/-/blob/master/community/sxmo-st/APKBUILD#L18) from the prepare function and the [subpackages definition](https://gitlab.alpinelinux.org/alpine/aports/-/blob/master/community/sxmo-st/APKBUILD#L11), i guess alpine does this because they handle documentation differently.

{% highlight bash %}
# Maintainer: dni <office@dnilabs.com>
pkgname=sxmo-st
pkgver=0.8.3.4
pkgrel=0
pkgdesc="St fork for Sxmo UI; supports scrollback, invert, and other patches"
arch=('x86_64' 'armv7h' 'aarch64')
url="https://git.sr.ht/~mil/sxmo-st"
license=('MIT')
depends=('ncurses')
makedepends=('make' 'fontconfig' 'freetype2' 'libx11' 'libxext' 'libxft')
provides=('st')
source=("$pkgname-$pkgver.tar.gz::https://git.sr.ht/~mil/sxmo-st/archive/$pkgver.tar.gz")
sha512sums=('a4cb72680e070d1703c4359618fa7ed36d5460496ab1320025f89809d7fa5387a6f9aae1e1143b261e57ea5f49d9b71bb68dad7653538b0238d2e50259029190')

build() {
  cd "$pkgname-$pkgver"
	make
}

package() {
  cd "$pkgname-$pkgver"
	make install PREFIX=/usr DESTDIR="$pkgdir"
	mkdir -p "$pkgdir"/usr/share/applications
	cat > "$pkgdir"/usr/share/applications/st.desktop <<-EOF
		[Desktop Entry]
		Name=st
		Comment=st is a simple virtual terminal emulator for X which sucks less
		Exec=st
		Terminal=false
		Type=Application
		Icon=gnome-xterm
		Categories=System;TerminalEmulator;
	EOF
}
{% endhighlight %}

#### building the packages
NOTE: sxmo isnt yet in Pine64-Arch repository so you need to create the below dir and put the PKGBUILD above inside there. for example:
{% highlight bash %}
git clone https://github.com/dreemurrs-embedded/Pine64-Arch
cd Pine64-Arch/PKGBUILDS/sxmo/smxo-st/
makepkg -s
{% endhighlight %}

#### next PKGBUILD sxmo-dwm
same as sxmo-st
{% highlight bash %}
# Maintainer: dni <office@dnilabs.com>
pkgname=sxmo-dwm
pkgver=6.2.12
pkgrel=0
pkgdesc="Dwm fork for Sxmo UI; supports volume-key hotkeys, swallow, keyboard, among other patches"
arch=('x86_64' 'armv7h' 'aarch64')
url="https://git.sr.ht/~mil/sxmo-dwm"
license=('MIT')
depends=('xorg-server')
makedepends=('make' 'freetype2' 'libx11' 'libxinerama' 'libxft')
provides=('dwm')
source=("$pkgname-$pkgver.tar.gz::https://git.sr.ht/~mil/sxmo-dwm/archive/$pkgver.tar.gz")
sha512sums=('842d93eb51204059e2f10c78dbf61f0239fef176d768b43a4d0b5f39a5ec452546e9fa77bc580a496f57f5a8f90a5576aa59a8964ffb0f86be2e454aec2b27c5')

build() {
  cd "$pkgname-$pkgver"
	make X11INC=/usr/include/X11 X11LIB=/usr/lib/X11 FREETYPEINC=/usr/include/freetype2
}

package() {
  cd "$pkgname-$pkgver"
	make PREFIX=/usr DESTDIR="$pkgdir" install
}
{% endhighlight %}



# Links

* [Github Repository Arch ARM Danctnix](https://github.com/dreemurrs-embedded/Pine64-Arch)
* [Arch ARM Danctnix Releases](https://github.com/dreemurrs-embedded/Pine64-Arch/releases)
* [Current Barebones Arch ARM](https://github.com/dreemurrs-embedded/Pine64-Arch/releases/download/20210613/archlinux-pinephone-barebone-20210613.img.xz)
* [sxmo repository](https://git.sr.ht/~mil/sxmo-build/tree/master/item/sxmo_build_all.sh)
* [sxmo-alarm repository](https://github.com/justinesmithies/sxmo-alarm)
* [alpine os sxmo packages](https://pkgs.alpinelinux.org/packages?name=sxmo*&branch=edge)
* [APKBUILD reference](https://wiki.alpinelinux.org/wiki/APKBUILD_Reference)
* [PKGBUILD reference](https://wiki.archlinux.org/title/PKGBUILD)
* [Creating Packages Arch](https://wiki.archlinux.org/title/Creating_packages)
* Matrix: #archmobile:kde.org
* IRC: #sxmo on irc.oftc.net
