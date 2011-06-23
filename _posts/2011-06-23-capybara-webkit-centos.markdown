
---
layout: post
title: Running Capybara-Webkit on CentOS 
author: Matt Robbins 
categories:
- testing 
- cucumber
- capybara 
- webkit 
---

[Capybara-Webkit](http://github.com/thoughtbot/capybara-webkit) is a headless driver for [Capybara](http://github.com/jinklas/capybara) that uses [Qt-Webkit](http://doc.qt.nokia.com/4.7/qtwebkit.html).  Although it does not actually create a head (and therefore should be a great deal quicker than say running firefox via Xvfb) it still needs an X11 server running for access to fonts etc, Xvfb will do the job just fine.

Running this on CentOS is complicated by a number of factors:

1. No official CentOS RPMs for Qt (or at least upto date ones)
2. No official CentOS RMPs for Ruby 1.8.7 
3. Need to setup and run Xvfb.

The following guide should get you well on the way...

### Pre-Requisites:

**Install Xvfb**

{% highlight console %}
yum install Xvfb Xorg
{% endhighlight %}

**Install Ruby 1.8.7 (as no CentOS RPM exists we must compile from source)**


{% highlight console %}
yum install -y gcc zlib zlib-devel  
wget http://ftp.ruby-lang.org/pub/ruby/1.8/ruby-1.8.7-p334.tar.gz  
tar xvf ruby-1.8.7-p330.tar.gz  
cd ruby-1.8.7-p330  
./configure --enable-pthread  
make  
make install  
{% endhighlight %}

You may need to add the ruby executable to your path.

**Install RubyGems 1.3.7**

Download: [rubygems-1.3.7.zip](http://rubyforge.org/frs/download.php/70697/rubygems-1.3.7.zip) and unzip
run `setup.rb `

**Install Qt**

Add the following repo using rpm:

{% highlight console %}
rpm -ivh http://software.freivald.com/centos/software.freivald.com-1.0.0-1.noarch.rpm
{% endhighlight %}

Then:

{% highlight console %}
yum groupinstall Qt4  
yum groupinstall Qt4-Devel
{% endhighlight %}

Add the qtmake binary to your path e.g.

{% highlight console %}
export PATH=$PATH:/usr/lib64/qt4/bin
{% endhighlight %}


### Testing your Xvfb setup

You can do this using Firefox...

{% highlight console %}
yum install firefox
{% endhighlight %}

Start Xvfb:

{% highlight ruby %}
Xvfb :1 -screen 0 1024x768x16 -nolisten inet6  (the no listen flag suppresses warnings related to no IPv6 support)
export DISPLAY=localhost:1.0 (exports the display)
DISPLAY=localhost:1.0 firefox (starts a headless firefox)
{% endhighlight %}

### Using the headless gem

It is probably easiest to use the [headless](https://github.com/leonid-shevtsov/headless) gem to manage your Xvfb server

For an example program using firefox and selenium-webdriver you could do something like the following:

{% highlight ruby %}
require 'rubygems'
require 'bundler/setup'
require 'headless'
require 'selenium-webdriver'

headless = Headless.new
headless.start

driver = Selenium::WebDriver.for :firefox
driver.navigate.to 'http://google.com'
p driver.browser
puts driver.title

headless.destroy
{% endhighlight %}

### Using Capybara-Webkit with Cucumber

Your env.rb could look something like this:

{% highlight ruby %}
require 'capybara/cucumber'
require 'capybara-webkit'
require 'headless'

headless = Headless.new
headless.start

Capybara.javascript_driver = :webkit
Capybara.run_server = false

at_exit do
  headless.destroy
end
{% endhighlight %}

