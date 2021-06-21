---
layout: post
title:  "running android apps with anbox on linux and pinephone"
categories: linux pinephone anbox android
---

# WIP
# introduction
TODO:

# pre installation
check if kernel modules are installed
{% highlight bash %}
modprobe -a binder-linux ashmem-linux
{% endhighlight %}
if they are not, install them with:
{% highlight bash %}
modprobe -a binder-linux ashmem-linux
{% endhighlight %}

# installation
download from AUR, i use trizen for that. package linux-zen-anbox included kernel modules if you already have them you dont need to install it.
{% highlight bash %}
trizen -S linux-zen-anbox
trizen -S anbox-image-houdini-rooted
trizen -S anbox-git
{% endhighlight %}

# configuration

# install an app

## encountered error
i got an unexpected error in the AUR public key is unknown when clone the git repostory, i solved it by using following commands
{% highlight bash %}
pacman-key --refresh-keys
gpg --keyserver keys.gnupg.net --recv-keys KEYNUMBER
{% endhighlight %}

[anbox]: https://anbox.io/
[arch wiki]: https://wiki.archlinux.org/index.php/Anbox
