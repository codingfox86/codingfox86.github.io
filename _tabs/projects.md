---
title: Projects
icon: fas fa-laptop-code
order: 2
---

{% assign projects = site.projects %}

{% for p in projects %}
- [{{ p.title }}]({{ p.url }})
{% endfor %}
