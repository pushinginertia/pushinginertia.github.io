---
layout: post
title: You can embed wicket:message in any HTML tag for easy internationalization in Wicket
tags: [Apache Wicket]
---

Here's a useful Wicket i18n tip to reduce the amount of code you need to write. Any HTML tag in your HTML template can have a `wicket:message` attribute added to it. This is especially useful in `<a>` and `<input>` tags.

For example, if you want a tooltip to appear when you hover the mouse over an `<a>` tag, you will use the `title` attribute, like so:

```html
<a
  href="#"
  title="Text to show on mouseover"
>click here</a>
```

However, this means that the user will see a tooltip in English regardless of their locale. The classic solution requires writing code in your Java component and using the `AttributeModifier` class to add a `title` attribute with a reference to `StringResourceModel` or `ResourceModel`.

Instead, you can use `wicket:message`:

```html
<a
  href="#"
  wicket:message="title:LinkTitleKey"
>click here!</a>
```

It's not necessary to include the attribute you are adding or replacing in your HTML template (in this example, `title=""` isn't necessary). Wicket will overwrite it if it exists and will add it if it doesn't exist.

The usage of `wicket:message` in this context is to specify the name of the attribute, followed by a colon, and then the resource key to look up (i.e., attribute:key). In this example, since we're adding or modifying the `title` attribute, the syntax is `title:LinkTitleKey`.

You might sometimes need to specify multiple attributes. An example would be in an `<input type="submit">` button, when you want to specify the text to display in the button (using the `name` attribute), but also define text to show in a mouseover tooltip (using the `title` attribute). Separate each attribute:key combination with a comma. For example:

```html
<input
  type="submit"
  wicket:message="name:InputName,title=InputTitle"
/>
```

Then, in your resource file, define a values for your keys:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM
  "http://java.sun.com/dtd/properties.dtd">
<properties>
  <entry key="LinkTitleKey">Text to show on mouseover</entry>
  <entry key="InputName">Submit</entry>
  <entry key="InputTitle">Mouseover text</entry>
</properties>
```

Your resource file must exist in the same source directory as your HTML template and Java component. For example, if your page is named MyPage, you will have `MyPage.java`, `MyPage.html`, and `MyPage.properties.xml`.
