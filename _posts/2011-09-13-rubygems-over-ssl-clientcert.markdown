---
layout: post
title: How to serve rubygems over ssl using client certification and Bundler
author: Matt Robbins 
categories:
- testing 
- ruby 
- bundler 
- bundler 
---

I was recently researching whether we could serve gems over a client certified ssl connection, so thought I would drop the results here in order that others can do the same if they need to.

Assuming you have have your Apache setup to do client certification and you have generated self signed certs for testing, then you can serve gems by doing the following:

1. Create a /gem directory in the document root
2. Run the index command from the directory above the /gem directory

{% highlight console %}
gem generate_index -d .
{% endhighlight %}

I am assuming you are using Bundler to manage your gem dependencies and that you invoke Bundler from the command line.  If this is the case then you need to do three things:

1. Monkey Patch rubygems to do the client certifaction
2. Invoke Bundler programatically after this patch has been applied (note, invoking it as a shell command won't work because this will start a new process and your monkey patch will not be effective)
3. Provide the command line options you would normally pass to Bundler as config in the file .bundle/config.  Note you can't provide these programatically because Bundler uses [Thor](http://github.com/wycats/thor) to parse command line options and Thor will freeze the options and prevent you from meddling with the resulting hash in your patch.

Here is the code I am using to invoke bundler:

{% highlight ruby %}
require 'rubygems/remote_fetcher'
require 'bundler'
require 'bundler/cli'

class Gem::RemoteFetcher
  def connection_for(uri)
    net_http_args = [uri.host, uri.port]

    if @proxy_uri then
      net_http_args += [
        @proxy_uri.host,
        @proxy_uri.port,
        @proxy_uri.user,
        @proxy_uri.password
      ]
    end

    connection_id = [Thread.current.object_id, *net_http_args].join ':'
    @connections[connection_id] ||= Net::HTTP.new(*net_http_args)
    connection = @connections[connection_id]

    if uri.scheme == 'https' and not connection.started? then
      require 'net/https'
      connection.use_ssl = true
      pem = File.read("/path/to/your/certwithkey.pem")
      connection.cert = OpenSSL::X509::Certificate.new(pem)
      connection.key = OpenSSL::PKey::RSA.new(pem)
      connection.verify_mode = OpenSSL::SSL::VERIFY_NONE
    end

    connection.start unless connection.started?

    connection
  rescue Errno::EHOSTDOWN => e
    raise FetchError.new(e.message, uri)
  end
end

Bundler::CLI.new.send('install')
{% endhighlight %}

Here is my config file which lives in the .bundle/config file within the project structure and which simulates the command line options '--binstubs --path vendor/bundle':

{% highlight yaml %}
---
BUNDLE_PATH: vendor/bundle
BUNDLE_BIN: bin
BUNDLE_DISABLE_SHARED_GEMS: "1"
{% endhighlight %}

By the way...if you know a cleaner way to do this...please let me know!!
