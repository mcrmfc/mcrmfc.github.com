---
layout: post
title: Selenium 141 - Alternatives to BrowserMob Proxy
author: Matt Robbins
categories:
- testing
- webdriver
- selenium
- browsermob-proxy
---

Selenium Issue 141
-------------------------------

The long story of [Selenium Issue 141](https://code.google.com/p/selenium/issues/detail?id=141) is not worth recounting here as it has been covered in depth by Selenium core committers [Simon Stewart](http://blog.rocketpoweredjetpants.com/2013/06/the-unix-philosophy-webdriver-and-http.html) and [David Burns](http://www.theautomatedtester.co.uk/blog/2013/the-tale-of-selenium-bug-141.html).

In a nutshell issue 141 is a hotly contested debate as to whether Selenium-Webdriver should provide public API for querying HTTP status codes and response headers.

I pretty much agree with the Selenium guys and can see how introducing this API would be the start of a very long and slippery slope for the project. However in the day to day work of an automation developer it is often essential to have access this information, for example most Web Apps have to do some kind of tracking which takes the form of a request for a 1x1 image with a query string containing the tracking key value pairs and we are frequently asked to validate these calls.


BrowserMob - the solution....maybe
----------------------------------

The conventional wisdom is that you should use the Selenium approved scriptable proxy [BroweserMob Proxy](http://bmp.lightbody.net/) to get around this problem.

Up until recently I followed this thinking and had used BrowserMob with reasonable success (see my earlier post about [using BrowserMob through a proxy](http://opensourcetester.co.uk/2012/01/04/browsermob-through-proxy/), however I had never had to use it in a critical CI pipeline and when I did things got a little less appealing.

There is no doubt that BrowserMob is a great project but using it via the [Ruby bindings](https://github.com/jarib/browsermob-proxy-rb) I found that all too often the Java process would not shut down cleanly after a test run (particularly if the test run failed for some other reason) meaning that on subsequent runs the port was locked and the proxy would not restart.

All these problems are solvable of course and I intend to keep using BrowserMob and contribute any fixes if I can.

However for the project I was on I needed something bullet proof which limited the amount of moving parts in my CI pipeline to capture and validate tracking calls from a mediaplayer.

Browser Extensions - an alternative approach...
------------------------------------------------

These trials took me to an alternative approach - browser extensions!

Starting with Chrome I knocked together an extension that uses the [chrome.webRequest](http://developer.chrome.com/extensions/webRequest.html) api to capture network requests and push them back into the page under test via local storage.  I then raise custom events which the page under test can register for.

Of course this assumes you are in control of the page under test, in my case I was testing a component on a test page so that was fine, I listened for the custom events and pushed them out into the DOM so I could scrape them using Selenium as normal.  If this is not the case for you (which is likely) I think you could still use this solution, for example by updating the extension to push the events into a global object which you could query by injecting JavaScript from Selenium.  Of course this is not an uber-clean but no solution in this area is!

I won't go into the code as you can have a [play for yourself](https://github.com/mcrmfc/chrome-netscraper), it turns out to be very simple and of course it is a bullet proof and fast way of capturing this information. 

But what about the other browsers?
------------------------------------

Of course this only addresses Chrome...to compete with BrowserMob we need to be able to cover as many of the other browsers supported by Selenium.

For this I used [Crossrider](http://crossrider.com/) which provides a limited api but allows you to generate extensions for Chrome, Firefox, IE and Safari.

You cannot be as sophisticated as you can using the Chrome APIs directly but you can still accomplish a fair amount.

Again feel free to have a play around with the simple [crossrider extension](https://github.com/mcrmfc/crossrider-netscraper) I wrote to scrape net requests.

Conclusions...
----------------

There is no 'right' way of dealing with the lower level aspects of web automation. In some cases using BrowserMob makes perfect sense and should be your go to solution.  However if you need a resilient way of capturing net requests, blocking net requests, accessing response codes or headers then a browser extension might just be the way to go.
