---
layout: home
title: Professor Bienz Blog
---

Welcome to my blog!

<ul>
  {% for post in site.posts %}
    <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
