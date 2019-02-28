---
layout: post
title: Deny requests by IP address on an Apache server
tags: [DevOps]
---

It's very unlikely that someone will legitimately access your web server by IP address in their web browser, and it's almost guaranteed to be a bot of some sort. To prevent access to your server by IP address, it's necessary to examine the `host` header in the HTTP request (`HTTP_HOST` in Apache) and then deny access if it evaluates to an IP address.

Add the following to `httpd.conf`.

```
RewriteCond %{HTTP_HOST} ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$
RewriteRule ^(.*)$ - [F,L]
```

For the [RewriteRule flags](https://httpd.apache.org/docs/trunk/rewrite/flags.html), F means forbidden and L means last rule. Technically, F implies L and httpd does this implicitly, but I prefer to be explicit.

This will throw a 403 Forbidden when trying to access the site by IP address.
