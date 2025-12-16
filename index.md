---
layout: default
title: "Home"
---

# Network Tools Lab

Welcome. This is a working log of a network software developer building and maintaining tooling around:

- Network automation (Cisco, infrastructure, observability)
- Python / FastAPI backends
- DevOps workflows (GitLab, Docker, Vault, Keycloak, etc.)

## Latest posts

<ul class="post-list">
  {% for post in site.posts limit:5 %}
  <li>
    <span class="post-meta">{{ post.date | date: "%Y-%m-%d" }}</span>
    <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
  </li>
  {% endfor %}
  {% if site.posts == empty %}
  <li><em>No posts yet. Ship something.</em></li>
  {% endif %}
</ul>
