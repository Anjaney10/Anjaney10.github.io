# Anjaney10.github.io

Bioinformatics & Computational Biology personal website built with the **Jekyll Chirpy** theme.

## Content structure

- `_posts/` — all blog posts in Markdown
- `_tabs/about.md` — author bio and research interests
- `_tabs/contact.md` — contact information
- `_tabs/tutorials.md` — tutorial listing page
- `_tabs/explainers.md` — explainer listing page
- `_tabs/tips.md` — tips and tricks listing page
- `_tabs/resources.md` — resources page
- `_tabs/blog.md` — all posts listing page
- `_includes/newsletter-signup.html` — reusable newsletter section

## Post template

Create a new file in `_posts/` with `YYYY-MM-DD-title.md`:

```md
---
layout: post
title: "Your Post Title"
date: 2026-06-14 09:00:00 +0000
categories: [Tutorials]
tags: [bioinformatics, python]
author: Anjaney
---

Your content in Markdown.

```

## Deployment

GitHub Pages deployment is configured in `.github/workflows/pages.yml`.
