---
title: "Migrate Images to COS"
date: 2021-04-05T15:37:55+08:00
slug: "migrate-images-to-cos"
type: post
tags:
  - cloud
categories:
  - Tech
---

Till now, Github has been the home of my images in this blog. It's reliable and cost-free, but it's not intended to host some images. Why I would choose this solution? The main reason was that *jsdelivr*, a free Github proxy provider, offers CDN nodes in China Mainland, making it pretty smooth to access the images hosted on Github in China.

As an amateur blogger, I always try to minimize the cost. Static site generator and Github Pages provide me with a possibility to build a blog without any burden, but media files have been a problem. For many reasons, foreign servers are extremely slow for users in China, and as a consequence, it is hard to efficiently deliver your images to visitors. However, through *jsdelivr*, we can use free CDN nodes in China for free and no  ICP  license is required. That's so convenient and sufficient that it has become one of the most popular methods to host the images for a personal blog in China.

But the problem is: they are not designed to be image hosting services. Github puts limitations on resources consumed and jsdelivr claims image hosting as abusing. It's not seriously considered in China as people have been accustomed to violating these rules more or less. The truth is that a personal blog won't take up much resource, while bloggers are not willing to pay much for it. Bloggers in China build their blogs mainly for sharing instead of earning. Sharing is voluntary work that some may enjoy, but when it costs, many of them just stop sharing. If the images are broken, they won't consider it an unacceptable lost. So hosting images on Github and jsdelivr has always been a good option.

I have been using this method for a long time and felt fine. Few images are needed in my posts, and the traffic is far from limitations. However, I still feel it a bit improper and unstable. So I now decide to move my images to specific services like OSS, S3, or something like that.