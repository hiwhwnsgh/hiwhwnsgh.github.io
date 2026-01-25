---
layout: default
title: Dev
permalink: /dev/
---

# Dev

<ul class="posts">
{% for post in site.categories.dev %}
  <li><span>{{ post.date | date: "%d %b %Y" }}</span> &raquo;
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
  </li>
{% endfor %}
</ul>
