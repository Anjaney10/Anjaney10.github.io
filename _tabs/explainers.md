---
icon: fas fa-lightbulb
title: Explainers
order: 4
---

{% assign explainers = site.posts | where_exp: "item", "item.categories contains 'Explainers'" %}

## Explainers

{% for post in explainers %}
- [{{ post.title }}]({{ post.url | relative_url }})
  - {{ post.date | date: "%b %d, %Y" }}
{% endfor %}
