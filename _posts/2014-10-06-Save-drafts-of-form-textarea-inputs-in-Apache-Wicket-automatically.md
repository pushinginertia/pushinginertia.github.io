---
layout: post
title: Save drafts of form textarea inputs in Apache Wicket automatically
tags: [Apache Wicket]
---

This article documents an Apache Wicket solution to automatically save drafts of user input in the background using ajax POST requests to the server.

The stateless nature of &lt;textarea&gt; form inputs introduces usability and reliability problems when users require a long time to fill them in. Consider some common scenarios, such as a user's web browser crashing after typing for five minutes or a user walking away and subsequently filling in the form input long after the server's session has expired, resulting in an access denied response when the form is submitted.

Either way, the work is lost and it's rare that a user would use the back button to copy the text and try again. Further, always-authenticated sites like Gmail have conditioned users to expect they can enter text into a form and wait perhaps hours before submitting it.

I chose to write a Wicket behavior that injects some client Javascript code into the page, which performs calls the server as a user types into one or more &lt;textarea&gt; fields. I took note of a few requirements and caveats before starting.

* It wouldn't be very efficient to send the form data to the server on each keypress made by the user, so some sort of timer must exist that either waits until the user stops typing or only sends the user's text periodically (e.g., every ten seconds).
* We want to avoid server calls when no changes have been made, but still call the server frequently enough as a keep-alive or heartbeat so that the user's session doesn't timeout (e.g., every five minutes).
* If a page has more than one form input supporting this functionality, we don't want to duplicate the server calls and therefore should attach the behavior to the form or page rather than the input itself.

There are a few strategies I could apply to work with the Javascript timer, but the simplest is to set an interval and save any unsaved drafts in the background at the end of each interval.

Consequently, I wrote client side code that sets a timer via the Javascript `setInterval(fn, interval)` function. When the page loads, it initializes an interval timer that then fires every `SAVE_DRAFT_INTERVAL` milliseconds. When the timer fires, it calls a custom function `saveDraft()`, which identifies the form inputs in which their text has changed since the last iteration and then performs an ajax POST request to the server with those changes. A timestamp is maintained of the last server callback, and if more than `KEEP_ALIVE_INTERVAL` milliseconds have elapsed since the last callback, an empty POST request is performed to ensure that the session doesn't expire.

This is all encapsulated inside my new class `FormInputSaveDraftBehavior`, which is instantiated by passing in the interval durations, a callback handler that fires when the client sends an updated draft to save on the server, and a collection of `FormComponent`s to enable automatic saving of drafts.

Let's look at an example.

```java
final TextArea messageBody =
    new TextArea<String>("messageBody", messageBodyModel);
form.add(messageBody);

form.add(
    new FormInputSaveDraftBehavior(
        new FormInputSaveDraftBehavior.SaveDraftCallback() {
            private static final long serialVersionUID = 1L;

            public void saveDraft(
                final FormComponent<String> component,
                final String value) {
                // validate that user is signed in if necessary
                if (!isSignedIn()) {
                    throw new RestartResponseException(LoginPage.class);
                }
                // ... logic to save the draft text ...
            }
        },
        Duration.seconds(15),
        Duration.minutes(5),
        messageBody));
```

This example initializes a form with a &lt;textarea&gt; input named `messageBody` and adds the new behavior to the form, passing `messageBody` as a reference. This is all that's needed to enable draft saves on the input. The duration parameters indicate that a background save will be made every 15 seconds if the input has changed in the previous 15 seconds and a keep-alive request will be made every five minutes if the user is inactive.
