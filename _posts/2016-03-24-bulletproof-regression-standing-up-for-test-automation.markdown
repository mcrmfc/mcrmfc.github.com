---
layout: post
title: Bulletproof Test Automation...
author: Matt Robbins
categories:
- testing
- selenium
- webdriver
- automation
---

# Bullet Proof Test Automation

I've been doing this test automation thing a while now, well over ten years in fact! I have worked across a number of industries using a multitude of tools and processes.

Throughout everything one consistent theme has emerged. Whilst people generally accept test automation is essential to building resilient deployment pipelines they often don't have much faith in the tests that get written, especially end to end tests...even more so UI tests.

The reason for this is simple, too often stakeholder's experience of test automation is that tests are unreliable, slow, lag behind the development cycle and despite their original purpose cannot be relied upon for assuring the quality of an imminent software release.

This problem gets worse the higher up the [test pyramid](http://martinfowler.com/bliki/TestPyramid.html) you go.

**_Discalaimer: yes a lot of these problems are solved by using the test pyramid to avoid over-relianance on end to end tests - but that's not always possible - not all projects are greenfield!_**

Here are some really interesting posts around this subject which I recommend reading if you haven't already:

* http://googletesting.blogspot.co.uk/2015/04/just-say-no-to-more-end-to-end-tests.html
* https://www.symphonious.net/2015/04/30/making-end-to-end-tests-work/
* http://www.alwaysagileconsulting.com/articles/end-to-end-testing-considered-harmful/
* http://devblog.songkick.com/2012/07/16/from-15-hours-to-15-seconds-reducing-a-crushing-build-time/

This quote sums up a lot of the issues we experience as authors of automated tests, it comes from the author of the [junit_merge](https://github.com/oggy/junit_merge) gem:

>"Of course, your test suite should pass 100% of the time, be free from nondeterminism, never modify global state, not rely on external services, and all those good things.

>But this is real life.

>Sometimes you don't have a spare week to diagnose intermittent failures plaguing your build. Or perhaps you're dealing with a legacy suite. Or you're relying on tools which offer no synchronization mechanisms, making you resort to sleeps which don't always suffice on a cheap, underpowered CI box. Or you're dealing with an integration suite that legitmately hits some external service over a flaky network connection.

Or perhaps more extreme is a great [tweet](https://twitter.com/jessitron/status/557921006543986688) from [Jessitron](https://twitter.com/jessitron)

>Selenium is like alcohol. Fun at first. You start to abuse it. Soon, weekly meetings about how much money and time you're wasting

### What's the answer?

I actually believe that as a discipline we consistently let ourselves down and have a tendency to blame outside forces for the fact the CI wallboard is more often red than green. Consequently faith is lost in end to end tests far to quickly and for legacy apps quite often teams end up accepting there is no answer, often I hear the phrase "well we are going to re-rewrite this application from scratch next year, then we'll do the testing properly".

I have successfuly implemented end to end UI test automation for numerous applications that didn't have a test pyramid to speak of yet which has helped teams release their software more frequently and with more confidence.

Here are some quick wins I think you can adopt to achieve this:

#### Infrastructure

Quite often test infrastructure causes problems, hands up if you run your tests on a snowflake CI server that was build by hand, has had numerous updates made by hand and consequently has drifted a long way from the setup developers have on their own machines?

* Build infrastructure in the cloud - it's a generalisation but experiance tells me that you can't move fast enough with on premise infrastructure, especially when dealing with test infrastructure that requires frequent updates and needs to contract/expand with demand.
* Use infrastructure automation tools such as HashiCorp's [Terraform](https://www.terraform.io/) to build out your cloud infrastructure - this way it's source controlled and repeatable.
* Use infrastructure automation tools to provision your hosts - [Chef](https://www.chef.io/chef/), [Puppet](https://puppetlabs.com/puppet/what-is-puppet), [Ansible](https://www.ansible.com/) - take your pick...but please don't allow people to easily update hosts by hand this way when the server crashes and burns we can rebuild back to last known good with a single click.
* [Jenkins](https://jenkins.io/index.html) - keep you build configuration simple, abstract to scripts where appropriate and consider backing your jobs with a dsl so they can be source controlled or at the very least source control the xml config (*ProTip: [Jenkins2](https://jenkins.io/2.0) will help when it finally arrives*)
* Consider using [Docker](https://docker.io) to build and run your automated test jobs - this will give you much tighter control over your dependencies and prevent you relying on system dependencies which may be transient.
* Use headless browsers where it makes sense e.g. [PhantomJS](http://phantomjs.org/) - you can achieve parallelism with less moving parts and latency.
* If you need access to "real" browsers then consider whether 3rd party providers such as [Browserstack](https://www.browserstack.com/) and [SauceLabs](https://saucelabs.com/) are going to work for you and bear in mind you will incur a financial and latancy cost (this largely depends on where you locate your cloud infrastructure relative to theirs), also try spinning up a simple grid yourself running say Chrome headlessly under XVFB on Linux, this might be cheaper and more performant!

#### Automation Code

Your test automation code is an application in its own right and should be treated with the care and respect it deserves.

* Architecture - try to avoid the 'big ball of mud' and break your automation code down into re-usable modules e.g an 'automation api' that drives the UI but doesn't concern itself with assertions or couple itself to a test runner (hint: think [Hexaganol Architecture/Ports and Adapters](http://alistair.cockburn.us/Hexagonal+architecture).
* Abstractions - as always be careful with anstractions but a Page Object Model is essential for UI testing, libraries exist that will do most of the heavy lifting for you, for example Webdriver's build in [Page Factory](https://github.com/SeleniumHQ/selenium/wiki/PageFactory) for Java or [Site-Prism](https://github.com/natritmeyer/site_prism) for Ruby.
* Use frameworks that handle basic wait scenarios - by that I mean page load and asynchronous JavaScript.
* Use an assertion library that produces sane output in error cases and if possible allows you to easily customise the output with your own messages, [RSpec](http://rspec.info/) for Ruby is a great example of such a library.
* If you are going to be testing UI applications on test environments (or maybe even production ones!) where timeouts or server errors occur frequently (trust me this is more common than people wish to admit) you will need to build mechanisms to handle this.  Re-runs are a classic use case here (re-running all failed tests again at the end of a test run), but I prefer to take a more hard line approach - detect those load failures and retry.
* Similarly you might need these mechanisms if you are testing an application using an asynchoronous architecture e.g. where events occur in the backend and take time to propogate through to API responses or the UI.
* Logging - it helps to use a good logging framework in your tests for two reasons, firstly it will make your log messages easier to read (you can have different levels, colours etc) and secondly you have the option to push your test log messages into the same log aggreagation tools that your app uses e.g. [Logstash](https://www.elastic.co/products/logstash).  This means that providing clocks are synced up you can tally up application log messages with test log messages which might make the cause of failure cases easier to diagnose.

This list could go on forever but if you follow the advise above you will be on the right track!

#### People

This might sound obvious but you won't be able to achieve any of the above without the right people.

Folks that end up being tasked with test automation (particularly UI automation) more often than not come from a manual testing background. This poses some challenges, they might not have a programming or Comp Sci background and they may not have had to wrestle with infrastructure let alone DevOps tooling before.

This isn't a problem per-se, but it is a problem if you expect these guys to build resilient test automation in the cloud that can form the basis of your CD pipeline without help, advice or assistance.

You will need to make an investment in bringing in the right peple either to do the work or train existing staff if you want to get the maxiumum value out of test automation.

I will be doing a workshop covering some of these topics at the upcoming [agile.delivery](http://agile.delivery/) conference, if you would like to discuss in more detail it would be great to see you there.
