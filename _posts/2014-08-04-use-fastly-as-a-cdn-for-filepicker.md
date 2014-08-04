---
published: true
layout: post
comments: true
related: false
description: Use Fastly as a CDN for Filepicker.io
title: Use Fastly as a CDN for Filepicker.io
---

We're using Filepicker.io for file uploads in the [Trabian CMS](https://www.trabian.com). Filepicker is awesome and saves us a ton of headaches. We wanted to put a CDN in front of their domain when we serve the files on client sites.

Our goals were:

1. Use a custom domain, with HTTPS
2. Speed up file requests
3. Cache file conversions to save time and money

Filepicker.io [suggests using CloudFront](https://developers.filepicker.io/docs/cdn/), which would have probably worked ok. However, Custom HTTPS is really expensive and we wanted a little more control. [Fastly](https://www.fastly.com/) is an awesome service, and gives us lots of control. Their support has been excellent so far and we are very please.

Here's how we configured Fastly to work with Filepicker.io.

## Domains
This is pretty simple, but we added our custom domain here.

## Hosts
We needed to setup 2 backend hosts. One for http and one for https. The address field is the important one, along with the appropriate condition.

```
Address: www.filepicker.io:80
Conditions:
  Name: is non ssl
  Apply If: !req.http.Fastly-SSL
  Priority: 10
```

```
Address: www.filepicker.io:443
Conditions:
  Name: is ssl
  Apply If: req.http.Fastly-SSL
  Priority: 10
```


## Settings
It is important to set the `Default Host` field to `www.filepicker.io` so that Fastly sends a host header.
Under Cache Settings, we have two special rules setup. One for 404s, which sets the TTL to 300s. The other extends the TTL beyond the Filepicker default to 43200s for all valid responses.

```
Name: 404s
TTL: 300 sec
State TTL: 300 sec
Action: cache
Conditions:
  Name: is 404
  Apply If: beresp.status == 404
  Priority: 10
```


```
Name: everything
TTL: 43200 sec
Stale TTL: 43200 sec
```

## Content
By default, Fastly won't cache any responses that have Cookies set. So it's important that we remove those headers.

Under headers, we have a rule:

```
Name: Remove Set-Cookie headers
Type: cache
Action: delete
Destination: http.Set-Cookie
Source:
Ignore If Set: 0
Priority: 10
```

Those are the configurations that we use in production. Hope this helps!

