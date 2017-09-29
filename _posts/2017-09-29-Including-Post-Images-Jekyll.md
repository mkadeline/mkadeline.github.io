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

{% highlight YAML %}
---
layout: post
title: "Including Post Images in Jekyll"
categories: programming web
excerpt_separator: <!--more-->
imageid: android.png
---
{% endhighlight %}

Using this property, it now becomes incredibly easy to include a header image. Now when listing your posts, you can reference this filename:

```HTML
{% raw %}
{% for post in site.posts %}
    <img class="card-img-top" src="/path/to/images/{{ post.imageid }}" alt="post image">">
    <span class="post-title-list"><a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a></span>
    <span class="post-excerpt">{{ post.excerpt }}</span>
{% endfor %}
{% endraw %}
```

With a little CSS magic, you can have your images comes out in a uniform shape and layout. This method can be extended to list specific articles with flags, attach banner images to the top of posts etc.