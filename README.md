# Anjaney10.github.io

Bioinformatics & Computational Biology personal website built with the **Jekyll Chirpy** theme.

## Quick start

1. Install Ruby and Bundler.
2. Install dependencies:
   ```bash
   bundle install
   ```
3. Run locally:
   ```bash
   bundle exec jekyll serve
   ```
4. Open `http://127.0.0.1:4000`.

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
title: "Your Post Title"
date: 2026-06-14 09:00:00 +0000
categories: [Tutorials]
tags: [bioinformatics, python]
author: Anjaney
---

Your content in Markdown.

```python
print("code blocks support syntax highlighting")
```
```

## Comments (Giscus)

Update the `comments.giscus` values in `_config.yml` with your Giscus repository and category IDs.

## Deployment

GitHub Pages deployment is configured in `.github/workflows/pages.yml`.
