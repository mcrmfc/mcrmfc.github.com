---
layout: post
title: All Hail Selenium Grid 2
author: Matt Robbins 
categories:
- testing 
- grid 
- webdriver 
- selenium 
---

<p class="info"><strong>NOTE:</strong> this post is based on the selenium-server-standalone-2.0.0 release.</p> 

So we finally have [Grid 2](http://code.google.com/p/selenium/wiki/Grid2), Grid 2 is a complete re-write of Selenium Grid and brings us the following features:

1. Support for Selenium-Webdriver (Selenium 2)
2. Full backwards compatibility with Selenium-RC (Selenium 1)
3. Significant optimisations both in terms of efficiency and functionality, the headline being you no longer need a 1:1 mapping betweeen RC instances and browsers which meant huge memory consumption.  Now a single node server can support all browsers for that node.

If you have never heard of Selenium Grid we are talking about the ability to run distributed parallel Selenium tests across multiple machines, physical or virtual and all the administrative functionality that you would expect to need to support that.

I have only just started looking at Grid 2 so this is going to be a very brief overview.  Hopefuly as I explore its functionality further i'll do some more involved posts.

The two most useful sources of information I have found thus far are:

1. [This](http://code.google.com/p/selenium/wiki/Grid2) page on the [Selenium Wiki](http://code.google.com/p/selenium/wiki)
2. This video from the [Selenium Conference](http://www.seleniumconf.com/videos) of a presentation by the authors...

<object width="640" height="385"><param name="movie" value="http://www.youtube.com/v/rjc51R8nI2Y&color1=0xb1b1b1&color2=0xd0d0d0&hl=de_DE&feature=player_embedded&fs=1"></param><param name="allowFullScreen" value="true"></param><param name="allowScriptAccess" value="always"></param><embed src="http://www.youtube.com/v/rjc51R8nI2Y&color1=0xb1b1b1&color2=0xd0d0d0&hl=de_DE&feature=player_embedded&fs=1" type="application/x-shockwave-flash" allowfullscreen="true" allowScriptAccess="always" width="640" height="385"></embed></object>

Pretty much everything you need to get started is on the [Wiki](http://code.google.com/p/selenium/wiki/Grid2) page but below are a few things that might be useful to clarify in order to get the 'out of the box' behaviour working.

**To start the admin server:**

{% highlight console %}
java -jar selenium-server-standalone-2.0rc3.jar -role hub
{% endhighlight %}

**To start the node server:**

{% highlight console %}
java -jar selenium-server-standalone-2.0rc3.jar -role webdriver -hub http://127.0.0.1:4444/grid/register -port 5555 (Webdriver)
{% endhighlight %}

{% highlight console %}
java -jar selenium-server-standalone-2.0rc3.jar -role rc -hub http://127.0.0.1:4444/grid/register -port 5556 (RC)
{% endhighlight %}

Note: the port numbers are not particularly significant but if you want to run Webdriver and RC nodes they must run on different ports.

Once this is running you should be able to browse to the admin console and see the status of your grid:

http://hostname:4444/grid/console

![grid2 setup](/images/grid_2_console.png)

**IMPORTANT** - By default the grid will start with a default set of browsers, 5 Firefox, 5 Chrome, 1 IE - you can change this configuration by supplying command line options when you start the node server, see the [Selenium Wiki](http://code.google.com/p/selenium/wiki/Grid2) for more details.

Alternatively you can supply a config file in json format e.g.

{% highlight javascript %}
{
"capabilities":
        [
                {
                        "browserName":"firefox",
                        "maxInstances":5
                },
                {
                        "browserName":"chrome",
                        "maxInstances":5
                },
                {
                        "browserName":"internet explorer",
                        "version":"7",
                        "platform":"WINDOWS",
                        "maxInstances":1
                }
        ],
"configuration":
        {
                "cleanUpCycle":2000,
                "timeout":30000,
                "proxy":"org.openqa.grid.selenium.proxy.WebDriverRemoteProxy",
                "maxSession":5,
                "url":"http://192.168.102.130:5555/wd/hub",

        }
}
{% endhighlight %}

Then start the node as follows:


{% highlight console %}
java -jar selenium-server-standalone-2.0rc3.jar -role webdriver -nodeConfig myconfig.json -hub http://127.0.0.1:4444/grid/register
{% endhighlight %}

Note: I found that when using Windows VMWare images you need to ensure the following:

1. Configure nodes with actual IP address as above (as opposed to localhost, even if running node on same machine as hub)
2. Windows Firewall is turned off (obvious but worth mentioning)

**Running Tests**

It does not matter whether you run tests from plain Selenium-Webdriver, Cucumber driving Selenium-Webdriver, Cucumber & Capybara using Selenium-Webdriver, the determining factor that will result in your tests running on the grid is the options and capabilities you set when configuring your driver, here is an example of some working sets of Capabilities that I have been using when running with Cucumber/Capybara/Selenium-Webdriver.

*IE*

{% highlight console %}
{:url=>"http://hostname:5555/wd/hub", :browser=>:remote, :desired_capabilities=>#<Selenium::WebDriver::Remote::Capabilities:0x47336dc2 @capabilities={:browser_name=>:"internet explorer", :version=>"", :platform=>"WINDOWS", :javascript_enabled=>false, :css_selectors_enabled=>false, :takes_screenshot=>false, :native_events=>false, :rotatable=>false, :firefox_profile=>nil, :proxy=>nil, :url=>"http://192.168.102.128:5555/wd/hub"}>, :http_client=>#<Selenium::WebDriver::Remote::Http::Default:0x17d1e01f @proxy=nil, @timeout=nil>}
{% endhighlight %}

*Firefox*

{% highlight console %}
{:url=>"http://hostname:5555/wd/hub", :browser=>:remote, :desired_capabilities=>#<Selenium::WebDriver::Remote::Capabilities:0x3ced3512 @capabilities={:browser_name=>:firefox, :version=>"", :platform=>"WINDOWS", :javascript_enabled=>false, :css_selectors_enabled=>false, :takes_screenshot=>false, :native_events=>false, :rotatable=>false, :firefox_profile=>#<Selenium::WebDriver::Firefox::Profile:0x1e64a937 @extensions={}, @secure_ssl=false, @load_no_focus_lib=false, @untrusted_issuer=true, @model=nil, @additional_prefs={"network.proxy.type"=>"1", "network.proxy.no_proxies_on"=>"*.noproxy.myhost.co.uk", "network.proxy.http"=>"proxy_host", "network.proxy.https"=>"proxy_host", "network.proxy.http_port"=>"80", "network.proxy.https_port"=>"80"}, @native_events=false>, :proxy=>nil, :url=>"http://hostname:5555/wd/hub"}>, :http_client=>#<Selenium::WebDriver::Remote::Http::Default:0x53133637 @proxy=nil, @timeout=nil>}
{% endhighlight %}

**Parallelism on the Client**

If you are using ruby to drive your tests this is most easily done using either the [parallel](https://github.com/grosser/parallel) gem or it's more featureful brother the the [parallel tests](https://github.com/grosser/parallel_tests) gem.

Essentially the parallel family of gems will allow you to run either parallel across cores and/or threads.  Obviously the choice will be determined by your execution environment e.g. if running in a CI queue your job may only have a single core therefore you are better to go for multiple threading also when all we are doing is feeding some code to a remote server that is going to run the tests then having multiple threads may seem more sensible.

**Conclusions**

So there you have it...I have bearly scratched the surface of what I believe Grid 2 can offer, from listening to the presentation by the authors there is mouch more to discover.  Hopefuly I will get a chance to bring some of this out in future posts.
