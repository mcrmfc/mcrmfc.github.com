
---
layout: post
title: Running Capybara with Selenium-Webdriver Android Driver 
author: Matt Robbins 
categories:
- testing 
- cucumber
- capybara 
- android 
---

The [Selenium-Webdriver](http://code.google.com/p/selenium/) project has developed a [driver](http://code.google.com/p/selenium/wiki/AndroidDriver) for Android which can be run both on the emulator or a physical device.  [This](http://code.google.com/p/selenium/wiki/AndroidDriver) page has pretty much all you need to get going, but I thought I would clarify a few things that confused me when I ws trying to get this all working through Cucumber and Capybara.

When setting up the emulator as per the instructions on the Selenium wiki, in order to generate a list of targets you need to ensure you have actually installed the relevant SDK packages.  To do this start the Android SDK and AVD Manager:

{% highlight console %}
~/android-sdk/tools/android
{% endhighlight %}

Which should bring up a screen similar to the one below, select 'Installed packages' and add some...then you'll have some targets to select!

![android packages](/images/android_package.png)

After you have followed all the other steps and started your emulator then you should be able to write your features and steps as normal (though there are some gotchas...see below).  You env.rb should look something like the one below, then run your tests through Cucumber as normal.

{% highlight ruby %}
require 'cucumber'
require 'capybara/cucumber'

Capybara.register_driver :selenium_android do |app|
  Capybara::Driver::Selenium.new(app, {:browser => :remote, :url => "http://localhost:8080/wd/hub"})
end

Capybara.current_driver = :selenium_android
{% endhighlight %}

Gotchas...

1. I found that I needed to start the Webdriver server (Jetty Server) on the emulator manually i.e. open the emulator then select to view applications and select 'Webdriver'
2. I also found that if I started the actual browser manually on the emulator this prevented tests from running and I received 'End of file reached (EOFError)' errors
3. In order to see tests running in the browser it seemed that you needed to have the Webdriver server application open and visible on the device emulator
4. Lots of the functionality you're used to using in Capybara may not work as expected, this is due to very limited support for X-Path and CSS selectors in the Android driver...it may actually be better to use the Selenium-Webdriver gem directly.
