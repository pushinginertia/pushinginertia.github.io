---
layout: post
title: Configure WiQuery to not import JQuery automatically
tags: [Apache Wicket]
---

I've been working on a Wicket project that requires JQuery to be present on every single page rendered by Wicket. I do this by including a reference to JQuery from my base page. However, when I add a wiquery component to a subclass of my base page, it automatically inserts a reference to JQuery as well – resulting in markup that instructs the browser to download `jquery.js` twice from two different places.

The markup generated by wiquery looks similar to this:

```html
<script
  type="text/javascript"
  src="wicket/resource/org.odlabs.wiquery.core.commons.CoreJavaScriptResourceReference/jquery/jquery-1.5.2-ts1308119960000.js">
</script>
```

To avoid duplicate JQuery references, we can disable the `autoImportJQueryResource` property in wiquery's configuration. There are two ways to do this.

## The hacky solution

The simplest way is also the hackiest way, but it gets the job done. Override `Application.validateInit()` and add the following lines. This must be done in the `validateInit()` and not the `init()` method, because Wicket will call `init()`, then it will initialize your third party components (including wiquery), and finally call `validateInit()`.

```java
@Override
protected void validateInit() {
    super.validateInit();
    final WiQuerySettings wqs = WiQuerySettings.get();
    wqs.setAutoImportJQueryResource(false);
}
```

The `get()` method above on line 4 will throw an exception if wiquery has not been initialized.

## A better solution

The "correct" way is a bit more involved. It requires creating a `wiquery.properties` file that points to a custom initializer that you define. This initializer class will set the property as desired. `wiquery.properties` must reside in your war file under `WEB-INF/classes`, or packaged in your application's jar file, depending on how you like to build your war file.

`wiquery.properties`

```
initializer: com.pushinginertia.wicket.listeners.WiQueryInitializer
```

`WiQueryInitializer.java`

```java
package com.pushinginertia.wicket.listeners;
 
import org.apache.wicket.Application;
import org.odlabs.wiquery.core.commons.IWiQueryInitializer;
import org.odlabs.wiquery.core.commons.WiQuerySettings;
 
/**
 * wiquery custom settings
 */
public class WiQueryInitializer implements IWiQueryInitializer {
    public void init(Application a, WiQuerySettings wqs) {
        wqs.setAutoImportJQueryResource(false);
    }
}
```

In addition to `autoImportJQueryResource`, you can also set a few other boolean properties:

* `autoImportJQueryUIResource` – defines whether to always import JQuery UI or rely on the application to do it
* `embedGeneratedStatements` – defines whether to generate JavaScript inline or as a virtual filename reference
* `enableResourcesMerging`
* `enableWiqueryResourceManagement` – overrides autoImportJQueryResource and autoImportJQueryUIResource
* `minifiedResources` – defines if JavaScript should be minified (good for production code to keep the size down but makes debugging more difficult in development)
