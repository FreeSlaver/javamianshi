---
layout: page
breadcrumb: true
show_meta: false
title: "JVM面试题"
subheadline: "垃圾回收器，垃圾回收机制，CMS，G1详解"
header:
image_fullwidth: "header/career.jpg"
permalink: "/jvm/"
---
<ul>
    {% for post in site.categories.jvm %}
    <li><a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>