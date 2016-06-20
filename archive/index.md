---
layout: default
title: Archive
---

# Posts
<ul class="posts">
  {% for post in site.posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
