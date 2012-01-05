---
layout: post
title: Using Selenium Webdriver and Browsermob Proxy through a corporate proxy
author: Matt Robbins 
categories:
- testing 
- capybara 
- webdriver 
- selenium 
- browsermob 
---

[Browsermob Proxy](http://opensource.webmetrics.com/browsermob-proxy/) proxy is a really useful tool that enables the user to effectively have an add-on api to 'out of the box' Selenium Webdriver which enables you to do things such as set header values, dump out har data etc.  There is also a Ruby binding maintained as a separate gem which can be found [here](https://github.com/jarib/browsermob-proxy-rb).

The problem I had was how to use this through/behind a corporate proxy, obviously it works by starting Firefox with a profile pointing to the browsermob proxy, which would then need to be told to navigate through a corporate proxy/firewall.

It turns out that the Java project does support this in Beta 4 and is documented [here](https://github.com/webmetrics/browsermob-proxy). Note this feature is not currently in the released binary (Beta 3) so you will need to 'git clone' the project and 'mvn install' to generate the jar file.

I then added a crude monkey patch which can be seen below, obviously when the next Beta of the Java code is released I can hopefuly submit a pull request for the Gem, which will be more elegant and better thought out, but for now if you need the hack here is the code:


{% highlight ruby %}
require 'rubygems'
require 'bundler/setup'
require 'selenium/webdriver'
require 'browsermob/proxy'

#Patch required for proxy support - note: proxy support only added in beta-4 (not yet released as binary) and the patch is required for the gem to use this
#functionality.  Once beta-4 is released (Java Code) then a pull request could be submitted for the Gem.
module BrowserMob
  module Proxy
    class Client
      def self.from(server_url)
        port = JSON.parse(
          RestClient.post(URI.join(server_url, "proxy?httpProxy=myproxyhost:80").to_s, '')
        ).fetch('port')

        uri = URI.parse(File.join(server_url, "proxy", port.to_s))
        resource = RestClient::Resource.new(uri.to_s)

        Client.new resource, uri.host, port
      end
    end
  end
end

server = BrowserMob::Proxy::Server.new("/Users/robbim02/dev/utils/browsermob-proxy-2.0-beta-3/bin/browsermob-proxy") #=> #<BrowserMob::Proxy::Server:0x000001022c6ea8 ...>

server.start

proxy = server.create_proxy #=> #<BrowserMob::Proxy::Client:0x0000010224bdc0 ...>

profile = Selenium::WebDriver::Firefox::Profile.new #=> #<Selenium::WebDriver::Firefox::Profile:0x000001022bf748 ...>
profile.proxy = proxy.selenium_proxy

driver = Selenium::WebDriver.for :firefox, :profile => profile

proxy.new_har "google"
driver.get "http://www.google.com"

har = proxy.har #=> #<HAR::Archive:0x-27066c42d7e75fa6>
har.entries.first.request.url #=> "http://google.com"
har.save_to "tmp/google.har"

proxy.close
driver.quit
{% endhighlight %}
