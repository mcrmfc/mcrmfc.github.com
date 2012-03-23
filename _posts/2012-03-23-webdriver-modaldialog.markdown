---
layout: post
title: Modal Dialog Present exception Selenium-Webdriver
author: Matt Robbins 
categories:
- testing 
- webdriver 
- selenium 
- capybara
- ruby 
---

Let me tell you an incredibly boring story...

We have a scenario where a user clicks a link on a test page, some JS runs creating a cookie, the page reloads and then the cookie is read by JS on the next page as it loads and then is deleted.

In my test I wanted to assert that the cookie was created and the only way I could do this using Webdriver was by popping up a JS alert on the test page when the link is clicked to delay the page reload and then using Webdriver read the cookies.

{% highlight ruby %}
#whilst the page reload is blocked by the JS alert
cookie = page.driver.browser.manage.cookie_named 'foo'
alert = page.driver.browser.switch_to.alert #using js alert on acceptance test page to stop execution and allow time to check presence of cookie
alert.send('accept')
{% endhighlight %}

This hack worked fine and served the purpose it was intended for....

That was until the guys at Selenium got all responsible and my tests started failing with webdriver throwing a 'Modal dialog present' exception.

I can't find anything in the [change logs](http://code.google.com/p/selenium/source/browse/trunk/java/CHANGELOG) but it looks like they have done this deliberately which makes sense...why would you want to interact with the page when a modal dialog is present????

Oh well...thought I would just mention it in case anybody else was wondering why this had happened!


