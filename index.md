---
layout: page
title: Home
permalink: /
---

## 👋 Welcome

I'm **Anjaney**, and this site focuses on beginner-friendly **bioinformatics** and **computational biology** learning.

### Research interests
- Genomics and transcriptomics workflows
- Reproducible computational pipelines
- Python, R, and Bash for bioinformatics
- Biological data analysis and visualization

{% include newsletter-signup.html %}

## Recent posts

<ul class="content ps-0">
  {% for post in site.posts limit: 5 %}
    <li class="mb-3" style="list-style: none;">
      <a href="{{ post.url | relative_url }}"><strong>{{ post.title }}</strong></a><br>
      <small>{{ post.date | date: "%b %d, %Y" }} · {{ post.categories | join: ", " }} · {{ post.tags | join: ", " }}</small>
    </li>
  {% endfor %}
</ul>
