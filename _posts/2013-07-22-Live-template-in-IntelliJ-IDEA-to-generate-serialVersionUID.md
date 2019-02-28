---
layout: post
title: Live template in IntelliJ IDEA to generate serialVersionUID
tags: [Best Practices, Configuration]
---

Live templates in IntelliJ are cool. They reduce the number of keystrokes you need for repetitive code.

One recurring action is adding the serialVersionUID value to a serializable class.

If you have the serialization inspections enabled (Project Settings > Inspections > Serialization Issues > Serializable class without 'serialVersionUID'), then IntelliJ will warn you and you can hit Alt+Enter on the class to insert the value.

However, this inserts a generated value and I prefer to start with a value of one and increment the value each time I make a change to the class that breaks its compatibility with previous versions.

My preferred solution is to define a new live template. Head over to Project Settings > Live Templates and then add a new template to the ‘plain’ group.

Assign the following values:

* abbreviation: svu
* enable the 'Java code' and 'Groovy' checkboxes
* description: private static final long serialVersionUID = 1L;
* template text: `private static final long serialVersionUID = 1L;`

Be sure to add a newline to the end of the string in the 'Template text' input so that a new line is also inserted.

Save your changes, and then any time you need to insert a `serialVersionUID` variable, type `svu` and press [Tab] for the live template to fire.
