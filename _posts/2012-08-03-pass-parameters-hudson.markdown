---
layout: post
title: Passing parameters to a downstream Hudson (Jenkins) job
author: Matt Robbins 
categories:
- testing 
- ci
---

I wanted to run an acceptance test in a downstream job.  However I needed to grab the value of a build parameter from the upstream job.  It turns out Hudson (Jenkins) does not support this 'out of the box'.

You can add a [plugin](http://wiki.hudson-ci.org/display/HUDSON/Parameterized+Trigger+Plugin), however this was going to cause a time delay in my environment as I did not have admin rights to do this.

So a thought struck me...trigger the downstream job from the shell in the upstream and use the REST API.

Now this is all fine and to be fair there is [reference](http://wiki.hudson-ci.org/display/HUDSON/Remote+access+API) on the Hudson wiki but it is not totally obvious.

You can't do a get with a query string it needs to be a POST with params, below is an example using curl.

{% highlight console %}
json="{\"parameter\": [{\"name\": \"NAMESPACE\", \"value\": \"test\"}, {\"name\": \"ENVIRONMENT\", \"value\": \"production\"}], \"\": \"\"}"
url="https://ci.mysite.co.uk/hudson/job/mysite-acceptance-tests/build"
curl -X POST $url -d token=FRAMEWORKS --data-urlencode json="$json" -k -E "/data/certs/mycert.pem"
{% endhighlight %}

You'll notice you need the security token in order to run this. This can be set in the upstream job 'Configure > Trigger Builds Remotely > Authentication Token'

The funny thing is I did all of this and then realised the version of curl on the Red Hat 5 system I was on didn't support --data-urlencode, luckily --data sufficed for my needs.
