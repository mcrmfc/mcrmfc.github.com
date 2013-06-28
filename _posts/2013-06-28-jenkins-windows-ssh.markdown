---
layout: post
title: Git Clone via SSH using Jenkins on Windows
author: Matt Robbins 
categories:
- ci
- jenkins
---

This blog post is nothing revolutionary but more of a note to self so that I never have to learn this painful lesson again!

My current team has Windows specific CI dependencies, hence I had no choice but to setup our Jenkins instance on Windows.  The cleanest way to do this is to use the [Jenkins Installer](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+as+a+Windows+service) which installs Jenkins as a Windows Service.

That works fine, but of course the first challenge you face is cloning your Git repo over SSH (the default for the Jenkins Git plugin).

I'll assume you have done the following:

* Installed [Git Client](https://code.google.com/p/msysgit/downloads/list)
* Created [SSH Keys](https://help.github.com/articles/generating-ssh-keys#platform-windows) and ensured that you can successfully clone a repo from your Git Bash (running as current user).

    Note: In theory you should not use passwordless keys as anybody obtaining your key will have access to your account.  However for Jenkins you cannot have a password as the Job would hang when attempting to clone.  From Git Bash this problem is solved using [ssh-agent](https://help.github.com/articles/working-with-ssh-key-passphrases#platform-windows) but this won't work in Jenkins as we are using the Git plugin and not the shell.

    Bottom line - for Jenkins use a passwordless key - you can mitigate the risk by having a dedicated Jenkins Git User and only allowing them pull access.

* Set up the Git plugin in Jenkins via 'Manage Jenkins' and point it to the Git cmd file on the system, as in the screenshot below:

![jenkins_git_plugin](/images/jenkins_git_plugin.png)


Now we get to the tricky part.  When Jenkins runs as a Windows Service it does not run under a normal user account, it runs under the "Local System Account".  Hence even though you have Git Clone working as the current user it will fail in Jenkins.

To solve this problem we need to put our private ssh key in the correct location for the Local System Account.

First of all we need to find out where this user's home directory is, since it will not exist under C:\Users...

* Download [PSTools](http://download.sysinternals.com/files/PSTools.zip)
* Extract to a conventiant location and using a command prompt navigate to that location (or add to path if you prefer)
* Start the PsExec tool by running:

{% highlight console %}
PsExec.exe -i -s cmd.exe
{% endhighlight %}

* Now we need to find out where this user's home directory is:

{% highlight console %}
echo %userprofile%
{% endhighlight %}

* For me this location was:

{% highlight console %}
C:\Windows\system32\config\systemprofile
{% endhighlight %}

* Now you can open Git Bash within this shell by running the following command:

{% highlight console %}
C:\Program Files (x86)\Git\bin\sh –login –i
{% endhighlight %}

* You can copy your existing private key into the correct location for the Local System User, for me this was something like the following:

{% highlight console %}
cd ~
cp /c/Users/matt/.ssh/id_rsa ~/.ssh/
{% endhighlight %}

* Finally you can test that you can successfully authenticate to Github as the Local System User:

{% highlight console %}
ssh -T git@github.com
{% endhighlight %}

* If all is successful you should see the usual message:

{% highlight console %}
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
{% endhighlight %}


All that's left to do now is test from Jenkins...Huzzah!
