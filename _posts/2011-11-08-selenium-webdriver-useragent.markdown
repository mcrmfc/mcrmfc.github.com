---
layout: post
title: Custom User-Agent using Selenium-Webdriver Remote and Ruby bindings/Capybara 
author: Matt Robbins 
categories:
- testing 
- webdriver 
- selenium 
- cucumber 
- ruby 
---

As far as I understand it the [Selenium-Webdriver](http://code.google.com/p/selenium/) project made an architectural decision not to provide an API for maniplulating headers when running 'in-browser' tests.  This implied that the only way of doing this would be to use an intermediate proxy such as [browsermob-proxy](http://opensource.webmetrics.com/browsermob-proxy/), which for most people might be fine.  However within my work infrastructure introducing such a proxy was not feasible.

So when I needed to do some 'in-browser' testing using different User-Agents I was delighted to find there is in fact a way around this...using [Chrome Switches](http://peter.sh/experiments/chromium-command-line-switches/).

You set these using the Webdriver capabilities when starting your tests, below is an example using the Ruby bindings directly and as an alternative using Capybara. This example is using the API for the client bindings directly (i.e. opening browser on your local machine)

{% highlight ruby %}
#starting the browser from the client bindings, setting a custom User-Agent
driver = Selenium::WebDriver.for :chrome, :switches => ['--user-agent=Mozilla/5.0 (PLAYSTATION 3; 3.55)']
#example using Capybara (0.4.1.2)
opts[:switches] = ['--user-agent=Mozilla/5.0 (PLAYSTATION 3; 3.55)']
Capybara::Driver::Selenium.new(app,opts)
{% endhighlight %}

The api is a little different when using the remote Webdriver...

{% highlight ruby %}
#starting the browser from the remote bindings, setting a custom User-Agent
remote_opts['chrome.switches'] = ['--user-agent=Mozilla/5.0 (PLAYSTATION 3; 3.55)']
remote_caps = Selenium::WebDriver::Remote::Capabilities.new(remote_opts)
driver = WebDriver.for(:remote, :desired_capabilities => remote_caps)
#example using Capybara (0.4.1.2)
remote_opts['chrome.switches'] = ['--user-agent=Mozilla/5.0 (PLAYSTATION 3; 3.55)']
remote_caps = Selenium::WebDriver::Remote::Capabilities.new(remote_opts)
opts[:desired_capabilities] = caps
Capybara::Driver::Selenium.new(app,opts)
{% endhighlight %}

Note: if your using Capybara 1 or greater I think the only difference would be the namespacing of the Selenium Driver...so something like:

{% highlight ruby %}
Capaybara::Selenium::Driver.new(app,opts)
{% endhighlight %}
