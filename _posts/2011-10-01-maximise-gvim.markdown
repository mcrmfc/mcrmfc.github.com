---
layout: post
title: How to maximise Gvim in Ubuntu
author: Matt Robbins 
categories:
- gvim 
- vim 
- ubuntu 
---

Something that has always troubled me is getting GVim to maximise when using Ubuntu..."WHATTTT" you say "that's easy...just add this to your .gvimrc"


{% highlight console %}
set lines=999
set columns=999
{% endhighlight %}

Well that did work - but it always moved Gvim into another workspace - one of those things that doesn't really matter, but never the less you just can't let go...it annoyed me everytime I opened GVim.

At last I have found a much better solution...one that I am sure any seasoned Ubuntu user knows about but which I had never heard of...[Devilspie](https://help.ubuntu.com/community/Devilspie/)

Roughly speaking this is what you do:

Install devilspie:

{% highlight console %}
sudo apt-get install devilspie
{% endhighlight %}

Create a script:

{% highlight console %}
cd ~/.devilspie
touch max_gvim.ds
{% endhighlight %}

Edit the script and add the follwing:

{% highlight console %}
(if
  (contains(window_name) "Vim")
  (maximize)
 )
{% endhighlight %}

Add devilspie to your startup applications and voila...maximised GVim!
