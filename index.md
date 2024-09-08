---
layout: default
title: Home
---

# Welcome to My SQL Server Blog!

Collection of articles about SQL Server

## Recent Blog Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <small>Published on {{ post.date | date: "%B %d, %Y" }}</small>
    </li>
  {% endfor %}
</ul>
