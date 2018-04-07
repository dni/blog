---
layout: post
title:  "creating qr codes and simple a website for an interactive art installation"
date:   2018-04-07 15:56:43 +0200
categories: web html art
---
A friend of mine, Patrick Fürli, is artists, he creates so called collage art, he takes pieces of magazine or paper, and rearranges them. He recently started creating animated gif files of his art and then he asked me: "do you have any good idea how to play this gifs with music in an interactive art installation which i can afford?". He had some kind of big installation in mind with multiple monitors, speakers, raspberry pies and other stuff.

1 or 2 days later i saw a qr code and this idea hit me. what about just generating qr code which you can place next to the original pieces. people at the vernisage could then scan it with their smartphones, which takes you directly to a website where the animated gif and a looping mp3 starts. i told him the idea and he likes it.

so i started thinking about how could i make this little website so he could as simple as possible.

my plan
---

1. use github pages for free webhosting
2. pre generate qr codes with custom urls, ex. http://dni.github.io/#1
3. the artist can upload 1.gif and 1.mp3 via github as many as he wants
4. create a simple overview for all qr codes
5. create some javascript which handles the urls and changes the files


qr codes
---
1. install the qr code nodejs module
{% highlight bash %}
npm i -g qrcode
{% endhighlight %}

2. pre generating arbritary mane qr codes as svg with this shell script
{% highlight bash %}
a=1
while [ $a -lt 101 ]
do
   qrcode -o qr/$a.svg "http://fuerli.dnilabs.com/#$a"
  a=`expr $a + 1`
done
{% endhighlight %}

html & css
---
{% highlight html %}
  <div id="contents" style="display: none;"></div>
  <audio id="audio" loop controls></audio>
  <a id="back" href="index.html" onclick="timeout()">Overview</a>
{% endhighlight %}

javascript
---

future plans
---
it's online, but for now there is only the first qr codes filled with sample data. but soon there should be more to see! one idea we had was to maybe implement a 1.txt with some metadata about the artwork.

you can view the source on [github][repo] and the live example of the [overview page][link1] and the example of the [artwork page][link2], be careful the audio my scare you :).

[repo]: https://github.com/dni/fuerli
[link1]: http://fuerli.dnilabs.com/
[link2]: http://fuerli.dnilabs.com/#1
