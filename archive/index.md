---
layout: default
title: Archive
---

<ul class="posts">
  {% for post in site.posts %}
    <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
        <em class="text-muted">{{ post.tags | sort | join: ", " }}</em>
    </li>
  {% endfor %}
</ul>
