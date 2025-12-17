---
layout: default
title: "Home"
---

# My Open Source Network Tools and Utilities

Welcome. This blog is my attempt to give back to the networking community.

The goal is to share what I’ve learned over the years with people who are just getting started, as well as those already in the field who want to level up, learn something new, or collaborate. I’ll be walking through tools and patterns I use in my day-to-day work, with posts tied directly to code and lessons in the repo so you can follow along end-to-end.

Topics I plan to cover include (but aren’t limited to):

- **Network automation** – multi-vendor environments (Cisco, Juniper, Alcatel, and others), infrastructure, and observability  
- **Python / FastAPI backends** – APIs and services that power internal network tools  
- **DevOps workflows** – Git-first practices, GitLab CI/CD, containerized deployments with Docker, and supporting services like Vault, Keycloak, and relational databases

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
