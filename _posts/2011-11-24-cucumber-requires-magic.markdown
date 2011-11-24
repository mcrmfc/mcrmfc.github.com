---
layout: post
title: Cucumber 'requires' option - autoload magic for your objects
author: Matt Robbins 
categories:
- testing 
- cucumber
- ruby
---

Once your [cucumber](http://cukes.info) projects get to a certain size you almost certainly start to think about moving away from monolithic steps files and toward a more object based approach.

This [blog post](http://blog.josephwilk.net/cucumber/page-object-pattern.html) describes how you might go about doing this and is definitely worth a read.

So, I wanted to have a project structure something like that depicted below:

{% highlight console %}
/~features/
/ /~examples/
/ / /~step-definitions/
/ / / /-steps.rb
/ / /-example.feature
/ /~support/
/ / /-env.rb/
/ / /~objects/
/ / / /-myobjects.rb
{% endhighlight %}

And the good news is that this worked fine when I ran my cukes as follows:


{% highlight console %}
cucumber -p default
{% endhighlight %}

However, when I ran with a specific feature it blew up as it failed to load my custom objects (myobjects.rb).

{% highlight console %}
cucumber -p default features/examples/example.feature
{% endhighlight %}

{% highlight console %}
uninitialized constant MyCustomObject (NameError)
{% endhighlight %}

So like every good hacker I started to delve down into the innards of the Cucumber runtime code, which whilst interesting was unnecessary had I read the help!!!

{% highlight console %}
-r, --require LIBRARY|DIR        Require files before executing the features. If this
                                     option is not specified, all *.rb files that are
                                     siblings or below the features will be loaded auto-
                                     matically. Automatic loading is disabled when this
                                     option is specified, and all loading becomes explicit.
                                     Files under directories named "support" are always
                                     loaded first.
                                     This option can be specified multiple times.
{% endhighlight %}

Note if you run a feature with tags then all code under /features will be pulled in, as when you run with no specificity.

So to always include all your code under /features, but still run a single feature - do the following:

{% highlight console %}
cucumber -p default --require features features/examples/example.feature
{% endhighlight %}
