---
layout: post
title:  "building the kernel for the pinephone"
categories: linux pinephone kernel
---


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

### if build fails
{% highlight bash %}
rm -r src/
{% endhighlight %}

# install kernel
{% highlight bash %}
sudo pacman -U linux-pine64..... pkg.tar.xz
{% endhighlight %}


# Links

* [Github Repository](https://github.com/dreemurrs-embedded/Pine64-Arch)
* Matrix: #archmobile:kde.org
