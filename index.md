---
layout: home
title: Professor Bienz Blog
---

<h2 class="text-center my-4">Blog Posts</h2>
<ul class="list-group">
  {% for post in site.posts %}
    <li class="list-group-item">
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

