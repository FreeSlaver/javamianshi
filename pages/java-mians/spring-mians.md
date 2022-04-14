---
layout: page
breadcrumb: true
show_meta: false
title: "Spring面试题"
subheadline: "Spring面试题,AOP,IOC，循环依赖，生命周期"
header:
image_fullwidth: "header/career.jpg"
permalink: "/spring-mians/"
---
<ul>
    {% for post in site.categories.spring-mians %}
    <li><a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>