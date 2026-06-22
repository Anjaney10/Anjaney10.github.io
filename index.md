---
layout: page
title: Home
permalink: /
---

I am **Anjaney**, a bioinformatician, and through this blog, I want to share interesting **bioinformatics** lessons I learnt along the journey.

### Research interests
- Genomics and spatia/single-cell transcriptomics workflows
- Reproducible computational pipelines
- Python, R, and Bash for bioinformatics
- Biological data analysis and visualization

## Recent posts

<ul class="content ps-0">
  {% for post in site.posts limit: 3 %}
    <li class="mb-3" style="list-style: none;">
      <a href="{{ post.url | relative_url }}"><strong>{{ post.title }}</strong></a><br>
      <small>{{ post.date | date: "%b %d, %Y" }} · {{ post.categories | join: ", " }} · {{ post.tags | join: ", " }}</small>
    </li>
  {% endfor %}
</ul>
