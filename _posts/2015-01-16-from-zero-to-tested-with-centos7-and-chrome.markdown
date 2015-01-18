---
layout: post
title: From zero to tested with Centos7, Chrome and Selenium
author: Matt Robbins
categories:
- testing
- webdriver
- selenium
- browsermob-proxy
---

If you are restricted to [Centos](http://www.centos.org/) in your CI environments and you need to run browser tests previous solutions include running in PhantomJS, an old version of Firefox under Xvfb or farming out to a Selenium Grid.

With Centos7 you now have another option as Chrome is fully supported on this OS.

You can easily run a headless (ish...[Xvfb](http://en.wikipedia.org/wiki/Xvfb)) Chrome browser.

Here I show you the bare minimum required to get from nothing to a working example.

Once you have it working you would need to consider repeatability e.g. provisioning the environment via something like Puppet, spinning it up in a Docker container or maybe just a simple script.

**Grab a Centos 7 VM**

{% highlight console %}
vagrant init matyunin/centos # a good community base image at time of writing
vagrant up
vagrant ssh
{% endhighlight %}

As root
-------

**Add google yum repo**

{% highlight console %}
cat << EOF > /etc/yum.repos.d/google-chrome.repo
[google-chrome]
name=google-chrome - \$basearch
baseurl=http://dl.google.com/linux/chrome/rpm/stable/\$basearch
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
EOF
{% endhighlight %}

As vagrant
----------

**Install dependencies**

{% highlight console %}
sudo yum install -y ruby ruby-devel gcc xorg-x11-server-Xvfb google-chrome-stable
gem install selenium-webdriver headless
version=`curl -s http://chromedriver.storage.googleapis.com/LATEST_RELEASE`
curl -s "http://chromedriver.storage.googleapis.com/$version/chromedriver_linux64.zip" > chromedriver.zip
unzip chromedriver.zip && sudo mv chromedriver /usr/local/bin/
{% endhighlight %}

**Create test script**

{% highlight console %}
cat << EOF > test.rb
require 'rubygems'
require 'headless'
require 'selenium-webdriver'
headless = Headless.new
headless.start
driver = Selenium::WebDriver.for :chrome
driver.navigate.to 'http://google.com'
puts driver.title
headless.destroy
EOF
{% endhighlight %}

Run test
--------

{% highlight console %}
ruby test.rb
{% endhighlight %}

Celebrate!
----------
