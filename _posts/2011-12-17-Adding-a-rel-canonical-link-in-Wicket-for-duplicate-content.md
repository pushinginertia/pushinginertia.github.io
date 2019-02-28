---
layout: post
title: Adding a rel=canonical link in Wicket for duplicate content
tags: [Apache Wicket]
---

Google recently introduced a new `<link rel="canonical">` tag to help the search engine identify duplicate content across domains or pages. I'm working on a development project that contains many subdomains under a single domain to present content specific to various geographical areas. Some of the administrative pages under each subdomain contain the same content (such as About Us, Privacy Policy, etc.).

The solution is very simple in Wicket by using a behavior. Create a new `Behavior` class as shown below and call it in the constructor of each page with duplicate content.

```java
/**
 * Generates a canonical link to a page:
 * e.g.,
 * <link rel="canonical" href="http://www.example.com/pagepath">
 */
public class CanonicalRefBehavior extends Behavior {
    private static final long serialVersionUID = 1L;
 
    private final String domain;
    private final String path;
 
    public CanonicalRefBehavior(String domain, String path) {
        this.domain = domain;
        this.path = path;
    }
 
    @Override
    public void renderHead(Component component, IHeaderResponse response) {
        final StringBuilder sb = new StringBuilder();
        sb.append("<link rel=\"canonical\" href=\"http://");
        sb.append(domain);
        sb.append('/');
        sb.append(path);
        sb.append("\"/>");
        response.renderString(sb.toString());
    }
}
```

To make use of this behavior, call it as follows in the page's constructor.

```java
add(new CanonicalRefBehavior(
    "www.example.com",
    getRequestCycle().getUrlRenderer().getBaseUrl().getPath()));
```

This example assumes that the path doesn't change between the page being rendered and its canonical reference. If the path differs, then appropriate changes can be made to the constructor call above.
