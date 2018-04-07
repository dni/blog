---
layout: post
title:  "Using multiple password storages with the unix password manager"
date:   2018-04-07 10:06:43 +0200
categories: unix security
---
In my new company we needed to securely share passwords over serveral admins and keep them in sync, i already used the unix password manager [Pass][pass]. So the problem arised that i need to keep my private passwords and my companies password seperate and also decrypt with multiple keys. It turns out this is quite easier than i thought.

The key is `git submodule`. Use the following lines to add a password submodule to pass and initialize it with the companies private key.

{% highlight bash %}
pass git submodule add git@github.com:dnilabs/pass.git subpass
pass init -p subpass PUBLIC-KEY-ID
{% endhighlight %}

Then you can insert new password and the right key will be automatically used within pass.

{% highlight bash %}
pass insert subpass/test-pass1
{% endhighlight %}

To commit a changes you need to commit through pass git and also push the submodule seperatly afterwards.

{% highlight bash %}
pass git add -A
pass git commit -am "add submodule changes"
cd ~/.password-storage/subpass
git push
{% endhighlight %}

[pass]: https://www.passwordstore.org/
