---
layout: post
title: Regex to identify an IE 6 user agent without blocking MSIE 8 or 9 users
tags: [DevOps]
---

I recently had a need to block all IE 6 users from a website. After doing some log analysis, it looked like nearly every IE 6 user agent was some sort of bot or undesirable crawler. The browsing patterns made it very obvious that the requests were not coming from a human, and sometimes the same IP address would send different user agent strings on each successive request.

There are hundreds of IE 6 user agent string variations, but an examination of some common ones quickly reveals a pattern:

```
Mozilla/4.0 (MSIE 6.0; Windows NT 5.1)
Mozilla/4.0 (MSIE 6.0; Windows NT 5.0)
Mozilla/5.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727)
Mozilla/4.0 (compatible;MSIE 6.0;Windows 98;Q312461)
Mozilla/4.0 (Windows; MSIE 6.0; Windows NT 6.0)
Mozilla/4.0 (Compatible; Windows NT 5.1; MSIE 6.0) (compatible; MSIE 6.0; Windows NT 5.1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)
Mozilla/4.0 (compatible; U; MSIE 6.0; Windows NT 5.1) (Compatible; ; ; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)
```

My initial regular expression to detect IE 6 and earlier was very simple and looked for the MSIE 6.0 seen in all of the above user agents.

```
.*MSIE [456]\\.[0-9].*
```

However, I noticed a number of legitimate IE 7 and 8 users sending user agents with "MSIE 6.0" embedded inside of the "MSIE 8.0" user agent string, triggering false positives using the above regex.

Some of the IE 8 user agents looked like:

```
Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; IPMS/5101ACA8-14FA1F2F282-00000016668C; User-agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; http://bsalsa.com) ; InfoPath.2; .NET4.0C)
Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0; GTB6.5; QQDownload 534; Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1) ; SLCC2; .NET CLR 2.0.50727; Media Center PC 6.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729)
Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0; Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1) ; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; InfoPath.3; Tablet PC 2.0)
```

Clearly, my initial regex isn't going to work. After some thinking and lots of unit testing, I came up with the following regular expression.

```
^Mozilla\/\d+\.\d+ \([^(]*MSIE [456]\.[0-9][^)]*\).*
```

Perfect.


