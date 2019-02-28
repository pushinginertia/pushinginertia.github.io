---
layout: post
title: A case for declaring your variables as final in Java
tags: [Best Practices]
---

I wrote a few months ago about Yoda conditions and using the final keyword. I recently came across a bug that would never have occurred if a final keyword was used.

Consider the following code. I've simplified it to the point that a static map structure could be used, but that was not the case in the original code. Can you spot the bug?

```java
public String demo(int input) {
    String retVal;
    switch (input) {
        case 1:
            retVal = "someValue1";
            break;
        case 2:
            retVal = "someValue2";
        default:
            retVal = null;
    }
    return retVal;
}
```

A `break` statement is missing in case 2, resulting in retVal being assigned null when input = 2. The compiler won't tell you anything is wrong, and if you haven't written a unit test for this case, you won't catch it that way either.

Now consider the following variation.

```java
public String demo(int input) {
    final String retVal;
    switch (input) {
        case 1:
            retVal = "someValue1";
            break;
        case 2:
            retVal = "someValue2";
        default:
            retVal = null;
    }
    return retVal;
}
```

Here, the double assignment will be detected at compilation time and the bug will (hopefully) never even make it into source control.
