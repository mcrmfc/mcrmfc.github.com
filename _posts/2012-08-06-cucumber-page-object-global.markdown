---
layout: post
title: Cucumber and the global scope problem - bring on page objects and the MGF pattern 
author: Matt Robbins 
categories:
- testing 
- cucumber
- cpaybara
---

There is no argument, [Cucumber](http://cukes.info) has added a new dimension to acceptance testing, Gherkin Features are **living executable requirements** and that for me is the benefit we should never loose sight of.

So you've clocked this and created a bunch of automated tests driven from you're features....but can too much Gherkin be a problem?

In my experience it can and it's something you need to keep a close eye on to avoid you're tests becoming as big a maintenance burden as the application code.

The Global Scope Problem
-------------------------

For a project with just a few features you can quite happily get away with a project structure that looks something like that below. All your steps in a single file containing the automation code for your features.

{% highlight console %}
/~features/
/ /~step-definitions/
/ / /-steps.rb
/ /-example1.feature
/ /-example2.feature
/ /~support/
/ / /-env.rb/
{% endhighlight %}

This is fine for a small project, but let's say you get above 10-15 scenarios, soon the steps file will become very big and you will start spreading steps over a few files with roughly representative names. Then you run headfirst into the 'Global Scope Problem'....

Cucmber has the concept of a ['World' object](https://github.com/cucumber/cucumber/wiki/a-whole-new-world), a kind of giant mixin which allows your Features, Steps and Support files to share context through the use of Ruby instance variables (the ones preceded by @ characters).  Whilst this is essential to the functioning of Cucumber it is also our enemy.  

As you're project grows this shared scope can cause chaos, an IDE or editor that understands Cucumber syntax will help, but ultimately you will have features mapping to steps which could be in any steps file and where state can be affected by any other step in any file via instance variables.

This can end up as mess of procedural spaghetti.

The Page Object solution and the MGF Pattern
---------------------------------------------

An excellent [blog post](http://blog.josephwilk.net/cucumber/page-object-pattern.html) by Joseph Wilk got me thinking seriously about this when I was working on a project which was being bitten hard by the global scope problem.  Integrating the Page Object Pattern into your Cucumber tests is really your only option for containing the problem of global scope.

At this point I am going to introduce what I am calling the MGF pattern, which stands for 'Model, Glue, Feature'.

* Model - You're page objects
* Glue - The step definitions
* Feature - The Gherkin Features

The Page Object Pattern will be familiar to most of those reading this blog so I won't go over it again.  However I will suggest the following 'Golden Rules'.  I have adapted these from Simon Stewart's [wiki page](http://code.google.com/p/selenium/wiki/PageObjects#Summary) on the Selenium site because as with most things he has it nailed:

1. PageObject methods return other PageObjects or Self (whenever a page load is triggered)...
2. Assertions should nearly always be in the Glue (step defs) not Page Objects (exception - 'correct location' check)
3. Different results for the same action are different methods ('save_expecting_error', 'save_expecting_submit')
4. Never call steps from steps (add helpers to objects if compounding steps are needed)
5. Have a hash of element locators in each PageObject (or register [custom locators](https://github.com/jnicklas/capybara/#xpath-css-and-selectors) with Capybara)
6. NEVER pollute test code with page internals i.e. no css xpath in test code

Page objects possess all the knowledge about the application under test, the Glue simply binds the objects to the Gherkin features.

If you follow this pattern I would expect you to end up with a project layout something like that below.

{% highlight console %}
/~features/
/ /~examples/
/ / /~step-definitions/
/ / / /-steps.rb
/ / /-signin.feature
/ / /-dashboard.feature
/ /~support/
/ / /-env.rb/
/ / /~objects/
/ / / /-signin.rb
/ / / /-dashboard.rb
{% endhighlight %}

Intergrating Capybara
----------------------

Many people including myself like using Capybara and so it is worth noting how to integrate this into your Page Objects. Because Page Objects will live outside the Cucumber 'World' then you have to options available:

1. Include the Capybara::DSL module in you're page objects
2. Pass around an instance of a Capybara Session

The advantage of the first method is that you only have to do this in a 'base' class which all other objects can inherit from and obtain those methods,  the advantage of the second method is that you could potentially pass in a mock if you wanted to unit test you're page objects (though the value of doing this is debatable and could fill an entire blog post).

Below is an example base class page object which may help get you started. In this example @session is an instance of an object which acts as a container for our current page and is the only variable that couples the step definitions (or Glue) it also acts as a way of storing data that needs to persist across a test run e.g. user.

{% highlight ruby %}

class GenericPage
  include Capybara::DSL
  include Capybara::Node::Matchers
  include RSpec::Matchers

  attr_accessor :url

  def initialize(session)
    @session = session
  end

  def correct_page?
    page.should have_css self.class::ELEMENTS['page check element']
  end

  def element_exists?(element)
    page.has_selector? self.class::ELEMENTS[element]
  end

  def current_url
    page.current_url
  end
end

class SigninPage < GenericPage

  LOCATION = 'http://myapp.com/signin'

  ELEMENTS = {
    'page check element'            => '.myapp-signin > h1',
    'page check value'              => 'Weclcome to MyApp',
    'username'                      => '#username',
    'password'                      => '#password',
    'save'                          => '#submit_button',
  }

  def initialize(session)
    super
  end

  def visit
    Capybara::visit LOCATION
  end

  def signin_expecting_success
    signin
    DashboardPage.new
  end

  def signin_expecting_error_using(unique)
    signin
    SigninErrorPage.new
  end

  private

  def signin
    fill_in ELEMENTS['username'], :with => @session.user.username
    fill_in ELEMENTS['password'], :with => @session.user.password
    click_on ELEMENTS['save']
  end
end
{% endhighlight %}

Conclusions
------------

Adding Page Objects and following the MGF pattern will help you combat the 'Global Scope' problem that can hit on large projects using Cucumber to drive automated tests.  It will mean changes can be isolated to single points in the code and add logical structure which will reduce the maintenance burden.

Update...
------------

I have just finshed reading some excellent posts on [Nathaniel Ritmeyer's blog](http://www.natontesting.com/) which elaborate on how we can [manage page objects](http://www.natontesting.com/2012/05/31/managing-your-page-objects/) thus tackling the issues of step coupling. He also maintains [SitePrism](http://www.natontesting.com/2012/07/30/siteprism-1-3/) which looks like an interesting dsl for page objects (and removes the need for the ugly constants I have in the examples above).

Anyway I think I will follow up this post in a couple of months and elaborate on the issues around coupling in our glue code and how best we can minimize what is one of the major problems with Cucumber.
