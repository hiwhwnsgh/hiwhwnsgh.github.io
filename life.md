---
layout: default
title: Life
permalink: /life/
---

# Life

<ul class="posts">
{% for post in site.categories.life %}
  <li><span>{{ post.date | date: "%d %b %Y" }}</span> &raquo;
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
  </li>
{% endfor %}
</ul>
