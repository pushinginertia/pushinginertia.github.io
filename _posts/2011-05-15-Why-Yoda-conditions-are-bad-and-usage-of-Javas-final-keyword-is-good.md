---
layout: post
title: Why Yoda conditions are bad and usage of Java's 'final' keyword is good
tags: [Best Practices]
---

I came across some code a while back that made me pause for a moment as I figured out its meaning. This code used something known as a Yoda condition, also sometimes referred to as Yoda notation. In a conditional block, the operands on an operator are reversed, so that the constant expression is listed first instead of second.

Let's take a look at this code snippet (assuming `s` is a string).

```java
if (null != s && 0 < s.length()) {
    // do something
}
```

Think about this for a moment and see how long it takes to figure out its meaning. In my case, I was looking at a method that was a few hundred lines long with many levels of embedded _if-else_ conditions. It was already rather challenging to trace through the logic, and seeing this notation sprinkled throughout the code added more mental complexity.

We speak and think in English using a 'subject-verb-object' grammar. For example, "(if) the grass is green." Or in the above example, "(if) s is not null and its length is greater than zero." It adds some mental gymnastics to think, "is green the grass," or, "not null is s and zero is less than its length." This usage just isn't how we think and it tends to slow down the thought process, and therefore, productivity, when reading code.

Let's examine why this notation was invented. It originates from back in the C days, when one could assign a value to a variable within the if clause. For example, `if (x = 0)` would actually assign the value zero to x. So, swapping the operands around prevented accidental assignment when the intent was to test that `x` equals zero. Perfect. This solution worked 15 years ago when we wrote our applications in C. However, modern languages such as Java and C# don't allow variable assignment in an if clause, and, in fact, require that the clause evaluates to a boolean. Modern IDEs such as IntelliJ will highlight this as an error as you write your code, long before you build it anyway.

Perhaps one might think that this helps prevent making accidental mistakes. Let's deconstruct this. First, note that the only place a Yoda condition actually works is on the equality operator (==) in languages that support variable assignment within the if clause. The above example would never assign a value of null to s because it's not testing if s is null, nor would it mutate its length because you can't assign a value to a method's return value (if what I written even makes sense).

Consider some best practices that remove any need for Yoda conditions.

First, reduce code complexity. It's generally agreed that methods should not exceed a dozen or so lines of code and code linters will typically raise warnings when functions are too long or contain too much [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity). It's likely that there is a lot of code duplication or there are subsets of logic that probably should be moved into their own methods. In the long term, this will help with code reusability if some other part of the system needs to call the same subset of logic. We've all seen someone take a long method and copy-paste it with a very small change to produce another long method, resulting in more unmaintainable code. Smaller subsets of logic discourage this practice. It also has the nice effect of removing the need to fix a bug in multiple places. I once interviewed someone who stated that methods should be concise enough that comments are not necessary to describe what they do. We hired him.

Perhaps we might want to consider the overhead of calling another method. These days, the efficiency bottlenecks are almost exclusively I/O based (e.g., database interaction) and an extra method call is unlikely to be noticed.

The second best practice is to write and run unit tests. This particular block of code had no unit tests. It couldn't. The method was too long to effectively write a unit test. Good unit test code coverage likely would have caught any coding errors.

Finally, consider the 'final' keyword in Java and how it relates to this problem. Java's final keyword is used to indicate that a variable's memory reference is immutable once assigned, unless it's a primitive type. Then the primitive type's value itself is immutable. This means that if a variable `s` of type String is assigned an instantiated String object, it cannot be assigned another String object. This is a good thing. It forces declaration of a new variable for each separate string being referenced. If you find you're assigning new values to the same string and it's not in a loop, you probably have some refactoring to do to move common logic into a method call.

Perhaps the Java grammar would have been better defined to default variables to final unless the variable is declared with a specific "mutable" modifier. It would seem reasonable to state that about 85% of variables can be marked as final in well-written code, preventing any possibility of reassignment. Note that I've stated above that 'final' makes the memory reference immutable. This means that you can still have a variable of type `Map` or `Set` and freely modify its contents, but you cannot assign a new map or set to the variable. In general, the only variables we can't reasonably mark as final are iterators, such as counters in a loop or possibly return values.

Unfortunately, C# does not have a construct similar to Java's final. The closest it has that mimic's Java's behaviour is the `sealed` keyword, but it's used at the class level and not for declaring variables.

In summary, usage of the final keyword will prevent a lot of accidental code problems and eliminates any possible need for Yoda notations. It also has the nice effect of forcing code into small, logical methods that are easy to unit test and easy to understand.
