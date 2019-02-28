---
layout: post
title: Python + peewee and OperationalError 'MySQL server has gone away'
tags: [Configuration]
---

It took hours to figure out why my python script using peewee yielded `OperationalError(2006, 'MySQL server has gone away')`, but I found the problem.

This error is very misleading and also results when MySQL responds with a `Packet Too Large` response on an INSERT statement. In my case, I'm inserting a very large 15-20 MB blob of data into a table, which triggers this error.

Once I figured out what the error was and what to look for, the solution is rather simple: set `max_allowed_packet` to a larger size in `my.cnf`.

