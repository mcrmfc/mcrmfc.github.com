---
layout: post
title: Qunit in the cloud - with Selenium Webdriver, RC and SauceLabs 
author: Matt Robbins 
categories:
- testing 
- qunit 
- webdriver 
- selenium 
- javascript 
---

I'll steer well clear of the debate around peoples preferences for Javascript unit testing frameworks and just settle on the fact that when all's said and done [Qunit](http://docs.jquery.com/Qunit) seems to be the one most commonly used.

Some of the devs in my team wanted an easy way to run Qunit tests against multiple browsers, and whilst I would have loved to offer them a local [Selenium Grid](http://code.google.com/p/selenium/wiki/Grid2) and a bunch of VMs I couldn't...the maintenance and cost overhead would have been too much.

However, I had been using the great service provided by the folks at [SauceLabs](http://saucelabs.com) for my Cucumber tests and thought that this would be a quick win for running Qunit tests against a bunch of browsers.  Essentially SauceLabs provide a Selenium Grid in the cloud...and provides support for both Selenim-Webdriver and 'old-school' Selenium-RC.

I had previously used a nice little project written one of the people at Sauce for running parallel Cucumber tests using Selenium-RC, which can be found [here](http://github.com/sgrove/cucumber_sauce).

Note this uses the excellent [parallel](http://github.com/grosser/parallel) gem to allow easy multi threading, and you may also want to check out [parallel_tests](http://github.com/grosser/parallel_tests) for a super easy way of running Cucumber tests across multiple cores.

Taking [cucumber_sauce](http://github.com/sgrove/cucumber_sauce) as my starting point I added support for Selenium-Webdriver and got rid of the Cucumber overhead (no need as all we are doing is hitting a single Qunit test page).

I ended up with [qunit-sauce](https://github.com/mcrmfc/qunit_sauce_runner), nothing clever and most of it was taken from the original project by Sean Grove (thank you Sean)...but it will certainly serve a useful purpose for my team...and hopefully squash a few extra bugs on the way!
