---
layout: post
title: Checking an element has focus using Selenium-Webdriver and Capybara 
author: Matt Robbins 
categories:
- testing 
- capybara 
- webdriver 
- selenium 
- cucumber 
---

Have you ever wanted to check which element currently has focus using [Capybara](http://code.google.com/p/selenium/) and/or [Selenium-Webdriver](http://code.google.com/p/selenium/)?  The API is not totally obvious so I thought I would put up one option here.

Using vanilla Webdriver:

{% highlight ruby %}
focused_element = driver.switch_to.active_element 
#now check for something on the found element
{% endhighlight %}

Using Capybara:

{% highlight ruby %}
focused_element = page.driver.browser.switch_to.active_element
#now check for something on the found element
{% endhighlight %}

Note: there is no direct api for this in Capybara as far as I am aware, hence the need to revert to the underlying driver.
