---
title: "Hello, Hugo!"
date: 2021-03-27T17:04:52+08:00
slug: "hello-hugo"
type: post
tags:
  - hugo
categories:
  - Other
---

## My Blog moved to Hugo

I've been working on moving my blog to Hugo, half a year after I had moved from Typecho to WordPress. Up to now, my blog has been migrated many times and maybe will never settle down.

To be honest, I'm satisfied with the WordPress solution in some cases, as the Argon Theme really provides me with an elegant look and powerful functions. Mathjax works well and I have comments, recent articles, related articles, TOC, and many more...

However, I finally decided to move, again. Why? That's a bit complicated, but let's see:

- WordPress requires a server and daily maintenance to be secure and up-to-date, which sometimes overwhelms me.
- WordPress stores its contents in a database, making it hard to migrate. More crucially, the data can be hard to preserve in the long run.
- WordPress is just **too heavy** for a personal blog whose articles are mainly made up of pure text. For me, the markdown syntax and an easy presentation are just enough.
- A server costs periodically, and I feel it kind of wasting to rent a server that runs 24 hours a day for a blog that may not be visited once a day.

In a word, I don't need a CMS for a simple personal blog, so I turned to static blog generators. They are simple, fast, cheap and reliable. What's more, they can render markdown syntax and I can export my article from Siyuan Note to them, which frees me from formatting the content meaninglessly. And they can be hosted on Github Pages, Vercel etc, which means that they can live long and well even I spend no time caring for it.

After comparing several popular static site generators, I chose Hugo. Hexo is good and has an active Chinese community, but for me, the slow generating speed may be unacceptable sometimes. Gatsby looks modern but too complicated for a high school student who knows nothing about React. Hugo is a good option as it is fast and simple.

Finally, my website was set up after a night and a day's work, hosting on Vercel. I spent some time migrating my important articles while abandoning many articles with less value.

Now that you can read this article, the website should be working safe and sound.  It can be accessed by [https://blog.codein.icu](https://blog.codein.icu). It is notable that the old WordPress Blog still runs well on [https://old.codein.icu/](https://old.codein.icu) as many articles haven't been migrated.