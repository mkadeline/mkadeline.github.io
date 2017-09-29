---
layout: post
title: "Including Post Images in Jekyll"
categories: programming web
excerpt_separator: <!--more-->
imageid: codemonkey.jpg
---

In setting up this site with Github pages and Jekyll, I was aiming for simplicity first and foremost. I really just wanted somewhere to work through new topics <!--more--> and technologies and provide a platform to get some advice. However, I still wanted it to look nice and in particular, add some colour through images. 

In particular, including some images in the post listings. Using liquid, it was relatively simple.

Jekyll provides the ability to add custom properties in the YAML front matter, so I simply included the image filename as a custom property. For example, for an android post I may include:

<script src="https://gist.github.com/mkadeline/3b6a36c01d5abee70b71ef543ae98c25.js"></script>

Using this property, it now becomes incredibly easy to include a header image. Now when listing your posts, you can reference this filename:

<script src="https://gist.github.com/mkadeline/13c668f5eb196a345480dc91be3d22bf.js"></script>

With a little CSS magic, you can have your images comes out in a uniform shape and layout. This method can be extended to list specific articles with flags, attach banner images to the top of posts etc.