---
layout: post
title: Live template in IntelliJ IDEA to generate static class-level Logger variable
tags: [Configuration]
---

This reduces the keystrokes needed to introduce a logger variable in your class to simply typing `log` and pressing [Tab] in the declaration block of your class.

Head over to Project Settings > Live Templates and then add a new template to the 'plain' group.

Assign the following values:

![IntelliJ Live Templates Logger configuration]({{ site.baseurl }}/images/live_templates_log.png)

* abbreviation: log
* description: private static final Logger LOG = LoggerFactory.getLogger(…);
* template text: `private static final org.slf4j.Logger LOG = org.slf4j.LoggerFactory.getLogger($CLASS_NAME$.class);`
* set the template to be applicable in Java declaration and Groovy declaration

The $CLASS_NAME$ variable will need to be defined. Press the _Edit variables_ button and set _Expression_ to `className()`.

![IntelliJ Live Templates Logger configuration - variables]({{ site.baseurl }}/images/live_templates_log_vars.png)

Save your changes, and then any time you need to insert a serialVersionUID variable, type `svu` and press [Tab] for the live template to fire.

