---
published: false
layout: post
comments: true
related: false
description: How to use custom OpenSSL with Nginx
---

Recently, I wanted to deploy TLSv1.1 and TLSv1.2 to our nginx servers. According to the nginx docs, this requires OpenSSL version 1.0.1 or higher. We're running CentOS 6.3 and the latest  version in the default respositories is `OpenSSL 1.0.0-fips 29 Mar 2010`.

I looked around for another way to upgrade our OpenSSL package and couldn't find anything that I trusted. (There were a few 3rd party repositories, but I didn't totally trust them.) Even the latest CentOS (6.4) doesn't have an upgraded OpenSSL package.

After a little more digging I found that nginx can be compiled with a custom OpenSSL. At first I assumed this meant you had to first install the custom OpenSSL and point the nginx isntaller to that, but it turns out it's even easier than that! This method doesn't touch your system OpenSSL (which was one of my original concerns).

All you need to do is download the OpenSSL source (from [www.openssl.org/source](http://www.openssl.org/source/)), extract the files, and add a config flag to the nginx build process. We were already [building nginx from source](http://nginx.org/en/docs/configure.html) so this was a simple change. Here's the config flag:

    --with-openssl=/path/to/openssl-1.0.1e
    
The configure command might look like this:

    ./configure --with-http_ssl_module --with-openssl=/path/to/openssl-1.0.1e
    
(Of course, you might need more flags for your specific installation.)

