---
icon: fas fa-blog
title: Blog
order: 7
---

{% assign all_posts = site.posts %}

## Blog

{% for post in all_posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
  - {{ post.date | date: "%b %d, %Y" }} · {{ post.categories | join: ", " }}
{% endfor %}
