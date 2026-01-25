---
layout: default
title: Interests
permalink: /interests/
---

# Interests

<ul class="posts">
{% for post in site.categories.interests %}
  <li><span>{{ post.date | date: "%d %b %Y" }}</span> &raquo;
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
  </li>
{% endfor %}
</ul>