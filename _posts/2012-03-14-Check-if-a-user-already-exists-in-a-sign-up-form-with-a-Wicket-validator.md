---
layout: post
title: Check if a user already exists in a sign up form with a Wicket validator
tags: [Apache Wicket]
---

A common use case during user sign up is to check if the user already exists, either by email address or by user name. Wicket makes this simple.

The following example demonstrates how to verify that an email address isn't already in use. It would be easy to write this to do a user name lookup instead.

There are three steps to take.

1. Write an email validator class.
2. Add the validator to the form input.
3. Define the error message to show the user.

Let’s start with the validator.

```java
/**
 * Ensures that the email address given during account
 * registration is not already in use by an active user.
 */
public class EmailNotInUseValidator extends
        AbstractValidator<String> {
    @Override
    protected void onValidate(
        IValidatable<String> validatable)
    {
        final String userEmail = validatable.getValue();
 
        final boolean emailAvailable =
            <insert your service call to check if
             the email address is used here>
        if (!emailAvailable) {
            error(validatable);
        }
    }
}
```

Now that we have a validator, it must be added to the email input. Add a new instantiation of the above validator to the (Email)TextField input.

```java
final EmailTextField email = new EmailTextField("email");
email.add(new EmailNotInUseValidator());
add(email);
```

Finally, define the error string. It's easiest to do this in `WicketApplication.properties.xml`, which resides in the same directory as your `WicketApplication` class. Internationalization is easy just by creating a separate resource file for each language.

You can define the error string in two ways. First, you can define a generic string that is displayed whenever the validator fires an error. Second, you can define a specific string when the validator fires an error on a specific input. I recommend always defining a generic string as a fallback, and then customizing as needed for specific inputs. In the former case, the key will be the name of the validator class (i.e., `EmailNotInUseValidator`), and in the latter, the key will be prefixed with the name of the input (i.e., `email.EmailNotInUseValidator`).

The following example defines a generic string and one specific to the above input.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <entry key="EmailNotInUseValidator">
        This email address is already in use.
        Please sign in to manage your account,
        or continue here with another email address.
    </entry>
    <entry key="email.EmailNotInUseValidator">
        This overrides the above string for any input
        with the name 'email.'
    </entry>
</properties>
```
