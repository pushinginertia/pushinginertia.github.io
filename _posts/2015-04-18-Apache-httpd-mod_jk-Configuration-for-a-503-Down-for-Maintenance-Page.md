---
layout: post
title: Apache httpd + mod_jk Configuration for a 503 "Down for Maintenance" Page
tags: [Configuration, DevOps]
---

This is a configuration I used to create a "Down for Maintenance" page with a 503 HTTP response code in httpd while I redeploy a Tomcat war file.

## JkUnMount a static resource

Since I have a dedicated server, my `mod_jk` configuration directs all traffic to Tomcat. In order to have Apache serve a static HTML file when I take the site down for maintenance, I need to use `JkUnMount` to prevent Apache from sending the request to Tomcat.

```
JkMount /* ajp13
JkMount / ajp13
JkUnMount /maintenance.html ajp13
```

## Configure Apache rewrite rule

The next step is to modify `httpd.conf` and set up a rewrite rule. This rule will serve the 503 response to all requests except those coming from the IP address 1.2.3.4, which would be my IP address that I can use for testing. The `maintenance.html` file should live in the `DocumentRoot` directory.

```
RewriteEngine on
RewriteCond %{REMOTE_ADDR} !^1\.2\.3\.4
RewriteCond %{ENV:REDIRECT_STATUS} !=503
RewriteRule ^(.*)$ /maintenance.html [R=503,L]
ErrorDocument 503 /maintenance.html
```

