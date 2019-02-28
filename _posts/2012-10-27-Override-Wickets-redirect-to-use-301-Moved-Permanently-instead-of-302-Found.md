---
layout: post
title: Override Wicket's redirect to use "301 Moved Permanently" instead of "302 Found"
tags: [Apache Wicket]
---

HTTP status codes *301 Moved Permanently* and *302 Found* are used to redirect a user's web browser to a different location. A user will never notice the difference if your server sends a 301 or 302, but sending a 302 can have negative SEO implications, such as PageRank not transferring correctly to the target page.

By default, an HTTP redirect issued by Wicket will send HTTP status code 302. Because the only user agent that will notice the difference between a 301 and 302 is a web crawler, there doesn't appear to be any harm in changing this behaviour to send a 301 by default.

If we look under the hood, Wicket's `ServletWebResponse.sendRedirect(String)` method calls `HttpServletResponse.sendRedirect(String)`, which is hard-coded to use HTTP status code 302. We'll need to customize Wicket's ServletWebResponse class and override the call to HttpServletResponse with our own redirect implementation. This solution works in Wicket 1.5.x (tested in 1.5.8) but the class has not changed in the 6.0.x versions and should probably work. Let me know if you use this solution on a 6.x deployment.

Let's begin.

First, we need to define a new subclass of `o.a.w.p.http.servlet.ServletWebResponse`. I've pasted my entire implementation below, which you can just drop into your project. `ServletWebResponse` performs different logic depending on if an ajax redirect is requested, and I've delegated to the Wicket implementation if an ajax redirect is requested so that nothing breaks. However, Wicket's HTTP redirect call to `HttpServletResponse.sendRedirect(String)` is gone, replaced with my implementation below.

I use some of Wicket's framework code to transform the requested redirect path into an absolute URL. The HTTP specification is a bit vague on whether the `Location` response header must have an absolute URL or if a relative path is acceptable, so I'm erring on the side of caution and always producing an absolute URL.

```java
/**
 * Overrides the default behaviour for redirects so that a 301 is issued instead of a 302. Good for SEO.
 */
public class ServletWebResponse301Redirect extends ServletWebResponse {
    // the superclass provides no visibility into this variable and it's needed to identify ajax requests
    private final ServletWebRequest webRequest;
 
    public ServletWebResponse301Redirect(
        ServletWebRequest webRequest,
        HttpServletResponse httpServletResponse)
    {
        super(webRequest, httpServletResponse);
        this.webRequest = webRequest;
    }
 
    @Override
    public void sendRedirect(final String url)
    {
        if (webRequest.isAjax()) {
            // delegate to the superclass if this is an ajax redirect
            super.sendRedirect(url);
            return;
        }
 
        disableCaching();
 
        final Url encodedUrl = localEncodeUrl(url);
        final UrlRenderer renderer = new UrlRenderer(webRequest);
        final String absoluteLocation = renderer.renderFullUrl(encodedUrl);
 
        final HttpServletResponse httpServletResponse = getContainerResponse();
        httpServletResponse.setStatus(HttpServletResponse.SC_MOVED_PERMANENTLY);
        httpServletResponse.setHeader("Location", absoluteLocation);
        httpServletResponse.setHeader("Connection", "close");
        flush();
    }
 
    private Url localEncodeUrl(final String url) {
        final String encodedUrl = encodeRedirectURL(url);
        if (encodedUrl.startsWith("./"))
        {
            /*
             * WICKET-4260 Tomcat does not canonalize urls, which leads to problems with IE
             * when url is relative and starts with a dot
             */
            return Url.parse(encodedUrl.substring(2));
        }
        return Url.parse(encodedUrl);
    }
}
```

Now that we have a custom subclass, let's tell Wicket to use it. Override `WicketApplication.newWebResponse` as follows.

```java
@Override
protected WebResponse newWebResponse(
    WebRequest webRequest,
    HttpServletResponse response)
{
    return new ServletWebResponse301Redirect(
        (ServletWebRequest)webRequest,
        response);
}
```

This completes the implementation.

To test, fire up your web application and try connecting to it via telnet. My example below is connecting directly to the servlet container (in this case, Jetty) on port 8080. The web application is running with the name mywebapp.

My example web application uses [LocaleFirstMapper](http://ci.apache.org/projects/wicket/apidocs/6.0.x/org/apache/wicket/examples/requestmapper/LocaleFirstMapper.html) to redirect requests for the root page to a subdirectory for the user's locale (e.g., redirecting a request for `/` to `/en`). You will see in the below transcript that the user agent is redirected with a status code of 301 (instead of the original 302) to the `/en` path.

```sh
$ telnet localhost 8080
Trying ::1...
Connected to localhost.
Escape character is '^]'.
GET /mywebapp/ HTTP/1.1
Host: mywebapp.com:8080

HTTP/1.1 301 Moved Permanently
Date: Sat, 27 Oct 2012 12:12:50 GMT
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Pragma: no-cache
Cache-Control: no-cache, no-store
Location: http://mywebapp.com:8080/mywebapp/en
Connection: close
Server: Jetty(6.1.26)

Connection closed by foreign host.
```
