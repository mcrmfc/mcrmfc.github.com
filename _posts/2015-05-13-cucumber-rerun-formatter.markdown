---
layout: post
title: Rerun failing tests with Cucumber - a solution for nondeterministic UI tests?
author: Matt Robbins
categories:
- testing
- cucumber
- selenium
---

The evolution of acceptance test automation, or at least my experience of it has been something like this:

_2005 - 2008_ : QA runs automated test on their machine pior to raising deployment ticket

_2009 - 2012_ : CI runs automated tests - downstream, everybody ignores them

_2012 - present_ : Continuous Delivery...UI tests broken, no deployments...now we have your attention!

So hopefully you have arrived at stage 3 and have done the following to ensure your UI tests are as resilient as possible:

1.  Followed Martin Fowler's [test pyramid](http://martinfowler.com/bliki/TestPyramid.html) and not implemented every conceivable system test via Selenium
2.  Implemented sane retry logic to find UI elements
3.  Isolated your UI using stub backends to guard against unexpected data
4.  Added spoonfuls of helpful debug logging to highlight issues

But your tests still appear now and again be nondeterministic and people are getting frustrated.

This can happen, UI testing is hard and no matter how much you defend against it peculiarities of the runtime environment can conspire against you.

In this instance it might be handy to have 'one more go' when you get some failures and Cucumber's rerun formatter allows this.

The Example
-----------

[This](https://github.com/mcrmfc/cucumber-rerun-example) is about the simplest example I could come up with.

The test will fail 50% of the time, allowing you to see the rerun kicking in on selected runs.

You can run the test using `bundle exec rake`

The test should be run for a second time if it fails and should terminate with an appropriate exit code.

You can see some examples of this working in the project's [Travis](https://travis-ci.org/mcrmfc/cucumber-rerun-example/builds) build.

Other Thoughts
----------------

Another useful thing might be to run any failing tests from previous CI runs first and you could probably adapt this approach to do just that!

Credit
------

I just thought I would illustrate with an example what others have already highlighted:

<http://blog.mattheworiordan.com/post/42587188567/automatically-retrying-failing-non-deterministic>

<http://blog.crowdint.com/2011/08/22/auto-retry-failed-cucumber-tests.html>

