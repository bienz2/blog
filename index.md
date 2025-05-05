---
layout: default
---

<ul class="list-group">
  {% for post in site.posts %}
    <li class="list-group-item d-flex justify-content-between">
      <span>• <a href="{{ post.url | relative_url }}">{{ post.title }}</a></span>
      <span class="text-muted small">{{ post.date | date: "%B %d, %Y" }}</span>
    </li>
  {% endfor %}
</ul>

