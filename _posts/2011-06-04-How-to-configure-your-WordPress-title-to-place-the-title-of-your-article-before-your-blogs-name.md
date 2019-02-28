---
layout: post
title: How to configure your WordPress title to place the title of your article before your blog's name
tags: [Usability and User Experience, WordPress]
---

_Update March 2019: This article was written in 2011 when this blog was hosted on WordPress and has since been moved to GitHub Pages._

Many WordPress themes default to rendering your blog's name first and the title of your blog or article last in the HTML title. For example, the title displayed on this blog defaults to the following:

Pushing Inertia » Blog Article Title

This results in every single page title showing static content first (your blog's name) followed by title of your article, and has two usability drawbacks:

* **Tabbed web browsing**: when one has multiple tabs opened in a web browser, they may only see the first two or three words of the title in the tab. If I had a number of tabs opened on this blog, I would only see "Pushing Inertia » ..." in each tab. Not very helpful for identifying the tab I want.
* **Search usability**: search engines usually display your title verbatim, but will only show the first 8-10 words (and even less on a mobile device). When someone is searching for a specific subject, the person is interested in finding a match to his or her query. You're better off getting as much of your title as possible into the search engine result to match what the reader is looking for, rather than filling that limited space with the name of your blog.

This problem can be solved by modifying the `header.php` file within your theme. This should be under `wp-content/themes/<theme>` or something similar.

Note that any changes you make to your theme will probably be overwritten and lost if the author of the theme releases a new version. I recommend keeping a list of modifications you made to the theme so that you can verify and re-apply them if necessary after a theme upgrade.

For my theme, my title is presented in two different ways, depending on whether I am on the home/front page or if I'm on a subpage. For the home or front page, the title is displayed as "Blog Name – Tagline" and for all other pages, it is displayed as "Page Title | Blog Name."

If you want the same behaviour as I've described, find the `<title>` tag in your `header.php` and modify it to look like this:

```php
<title><?php
if (is_home() || is_front_page()) {
    bloginfo('name'); echo ' - '; bloginfo('description');
} else {
    wp_title('|',true,'right'); bloginfo('name');
}
?></title>
```

Note that I am calling both the `is_home` and `is_front_page` functions. If you have configured your copy of WordPress to display a static page instead of the blog as the front page (Settings » Reading » Front page displays » A static page) then you'll need both of these checks.

If you'd like to change things up a bit, I recommend reading the WordPress reference guide on the `wp_title` function.
