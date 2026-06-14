---
icon: fas fa-wand-magic-sparkles
title: Tips & Tricks
order: 5
---

{% assign tips = site.posts | where_exp: "item", "item.categories contains 'Tips'" %}

## Tips & Tricks

{% for post in tips %}
- [{{ post.title }}]({{ post.url | relative_url }})
  - {{ post.date | date: "%b %d, %Y" }}
{% endfor %}
