---
layout: post
title: Execute asynchronous JavaScript from Selenium
author: Matt Robbins
categories:
- testing
- selenium
- webdriver
- javascript
---

When implementing end user acceptance tests in the browser using [Selenium-Webdriver](https://code.google.com/p/selenium) it's a good idea to follow these rules:

1. Wherever possible test the site from the perspective of the user i.e. with no access to the internals of the page
2. If you really need some data exposed from JavaScript and made available to the test (that the user wouldn't see) - have some kind of 'debug' mode where JavaScript events are logged to a debug JS object in the page which you can query from Webdriver using synchronous `execute_script` calls.

Occasionally however it might be that you can't add debug objects to the page (it's not under your control for some reason) and perhaps to know some asynchronous event has occurred it would be useful to, for example, register for an event.

Until recently I didn't think this was possible from Selenium...however I was completely wrong!

I stumbled across this [blog post](http://blog.jthoenes.net/2013/08/16/waiting-for-a-javascript-event-with-seleniumcapybara/) which shows you how to do it.

I thought it was worth expanding a little bit on this with some working examples and more links.

It turns out there is a method called `execute_async_script` which exists in most language bindings.  In the JavaDoc it's also well documented with [example scripts](http://selenium.googlecode.com/git/docs/api/java/org/openqa/selenium/JavascriptExecutor.html).

I have converted one to Ruby so you can prove to yourself this really works. If you run this you should observe the client code blocking until the asynchronous JavaScript has returned.  The Selenium method has an implicit callback (accessed via `arguments[arguments.length - 1]`) which we pass as the first argument to the asynchronous `setTimeout` JavaScript function. The Benchmark result proves this behaves as expected.

{% highlight ruby %}
require 'selenium-webdriver'
require 'benchmark'

driver = Selenium::WebDriver.for :firefox
driver.navigate.to "http://www.example.com"
driver.send(:bridge).setScriptTimeout(5000) # needs to be > setTimeout value

result = Benchmark.realtime do
  driver.execute_async_script( "window.setTimeout(arguments[arguments.length - 1], 4000);")
end

fail "Times do not match!" unless result.round(0) == 4
driver.quit
{% endhighlight %}

Note that the `setScriptTimeout` method exists in the [bridge](https://github.com/SeleniumHQ/selenium/blob/master/rb/lib/selenium/webdriver/remote/bridge.rb#L144) class and so it's not accessible via the public API in the Ruby Bindings (hence the `send` hack). This millisecond value must be higher than the time you think your callback will return in otherwise the method will return and report a timeout.

For a more tangible example here we register for a custom JavaScript event and accessing some of the event data...you can copy the html to a separate file and run it on your own machine.

{% highlight ruby %}
<<-example
<html>
  <head>
    <style>
      #green-box {
        height: 50px;
        width: 50px;
        background-color: green;
      }
    </style>
    <script>
      window.onload = function() {

        greenBox = document.getElementById('green-box');

        var myEvent = new CustomEvent("green-box-click", {
          detail: {
            username: "matt",
          },
          bubbles: true
        });

        window.addEventListener("green-box-click", function(e) {
          document.getElementById('status').textContent = 'clicked by:' + e.detail['username'];
        })

        greenBox.onclick = function() {
          greenBox.dispatchEvent(myEvent);
        }
      }
    </script>
  </head>
  <body>
    <div id='green-box'><span id='status'>click me</span></div>
  </body>
</html>
example

require 'selenium-webdriver'

driver = Selenium::WebDriver.for :firefox
driver.navigate.to "file:///home/matt/workspace/selenium-debug/test.html"

driver.send(:bridge).setScriptTimeout(5000)

puts driver.execute_async_script(
  "var callback = arguments[arguments.length - 1];
    window.addEventListener('green-box-click', function(e) {callback(e.detail['username'])});"
)

driver.quit
{% endhighlight %}

Hopefully this has given some more insight into executing asynchronous JavaScript from Selenium, certainly not something you would want to do unless you really had to but worth knowing about.
