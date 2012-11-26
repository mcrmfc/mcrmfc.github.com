---
layout: post
title: Setting two space indent for Gherkin Features in Vim
author: Matt Robbins 
categories:
- testing 
- vim
- cucumber
---

Users of Vim (or GVim, Macvim etc) who write [Gherkin Feature files](https://github.com/cucumber/cucumber/wiki/Gherkin) will know that for most installations (on Mac certainly) you appear to have auto-magical support for Gherkin Feature file syntax i.e. nice colours, auto format etc.

This is possible because the distribution of Vim installed on the machine, or which you installed comes shipped with Tim Pope's excellent [vim-cucumber plugin](https://github.com/tpope/vim-cucumber).

If your Vim does not ship with this you should most certainly install it right away!

I have always auto-indented using gg=G and never really thought too much about the indent level.  It turns out that presently the vim-cucumber plugin does not specify this (although I note that it is commented out in the source at the bottom of the [syntax file](https://github.com/tpope/vim-cucumber/blob/master/syntax/cucumber.vim))

My vimrc defines this globally as follows:

{% highlight console %}
set expandtab
set tabstop=4
set shiftwidth=4
{% endhighlight %}

So of course I was getting 4 level indent in my feature files.

The current team I am working with was not keen on this and I can see their point it does seem excessive, 2 would seem more than enough and in fact is what is used in the examples on [cukes.info](http://cukes.info)

So how best to do this?

One option would be to navigate to the file relevant to your Vim install location:

{% highlight console %}
#Examples
/Applications/MacVim-snapshot-64/MacVim.app/Contents/Resources/vim/runtime/syntax/cucumber.vim
/usr/share/vim/vim73/syntax/cucumber.vim
{% endhighlight %}

And change this line:

{% highlight console %}
" vim:set sts=2 sw=2:
{% endhighlight %}

To this:

{% highlight console %}
set sts=2 sw=2
{% endhighlight %}

This is a hack though (when you reinstall vim this change would be lost) and will need to be done for all copies of vim on the machine.

The answer is to use Vim's ftplugin functionality.

First of all you need to switch this on in your vimrc, adding the following:

{% highlight console %}
filetype plugin indent on
{% endhighlight %}

The you have two options, firstly you can create a file '~.vim/after/ftplugin/cucumber.vim' and add the following:

{% highlight console %}
setlocal expandtab
setlocal shiftwidth=2
setlocal softtabstop=2
{% endhighlight %}

Or add the config directly into your .vimrc:

{% highlight console %}
au FileType cucumber setl sw=2 sts=2 et
{% endhighlight %}

This is obviously common knowledge to experienced Vim users but may be useful for anybody out there who is relatively new to Vim and wants to customize syntax on a per language basis in a clean way.
