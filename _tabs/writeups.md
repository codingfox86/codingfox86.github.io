---
title: Writeups
icon: fas fa-flag
order: 3
---

All writeups (TryHackMe and more).

{% assign writeups = site.posts | where_exp: "post", "post.categories contains 'Writeups'" %}

{% for post in writeups %}
- [{{ post.title }}]({{ post.url }})
  <small>{{ post.date | date: "%Y-%m-%d" }}</small>
{% endfor %}
