---
layout: default
title: "Blog"
permalink: /blog/
---

# Blog

All posts, newest first.

<ul class="post-list">
  {% for post in site.posts %}
  <li>
    <span class="post-meta">{{ post.date | date: "%Y-%m-%d" }}</span>
    <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
  </li>
  {% endfor %}
  {% if site.posts == empty %}
  <li><em>No posts yet. Time to automate something.</em></li>
  {% endif %}
</ul>
