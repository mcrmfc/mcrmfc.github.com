---
layout: post
title: 3DES Zeros Padding with Ruby and Openssl
author: Matt Robbins 
categories:
- testing 
- ruby
---

Although many encryption libraries exist for Ruby (and here I am talking 'C' Ruby in the main) you will most typically want to use [Openssl](http://openssl.org).

Recently I needed to encrypt a simple JSON string using 3DES (ECB mode).  

IMPORTANT - You should never use ECB mode it is insecure and highly prone to brute force attacks....but I had no choice this is what the external service required.

It all seemed so simple and my initial script looked something like this:


{% highlight ruby %}
require 'openssl'
require 'base64'
require 'json'

#CIPHER
des = OpenSSL::Cipher::Cipher.new('des-ede3')
des.key = 'yourchosendeskey' 

#ENCRYPTION
des.encrypt
edata = des.update('{"somekey":"somevalue"}') + des.final
b64data = Base64.encode64(edata).gsub("\n",'') #remove EOL insterted in Base64 encoding
{% endhighlight %}

Although this worked perfectly in my local test harness the external PHP service failed to decrypt the data.  The reason for this is all to do with padding.

Certain encryption schemes (such as 3DES) are known as ['Block Cipers'](http://en.wikipedia.org/wiki/Block_cipher) and operate on a fixed block size i.e. in order for the data to encrypt/decrypt successfully it must be presented as a multiple of the required block size (in this case 8 bytes).

Openssl ensures this by inserting PKCS#5 padding, this padding scheme is preferred because it can be easily be distinguished from the actual data and allows for basic integrity or password checking.

Sadly (and possibly unsuprisingly) the PHP library I was integrating with did not use Openssl and instead used MCRYPT which uses 'Zeros Padding' and not PKCS#5.

Openssl does not support Zeros Padding out of the box but you can do this yourself by telling Openssl not to pad the data and ensuring the data is a multiple of the block size by adding '/0' characters at the end of the data, the code looks something like this:

{% highlight ruby %}
#ENCRYPTION
block_length = 8
des.padding = 0 #Tell Openssl not to pad
des.encrypt
json = '{"somekey":"somevalue"}'
json += "\0" until json.bytesize % block_length == 0 #Pad with zeros
edata = des.update(json) + des.final 
b64data = Base64.encode64(edata).gsub("\n",'')
{% endhighlight %}

IMPORTANT - you should choose a much better encryption scheme and padding mechanism, but if you really have to integrate with some 'interesting' PHP this may help!
