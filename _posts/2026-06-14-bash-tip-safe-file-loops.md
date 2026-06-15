---
layout: post
title: "Tip: Safer File Loops in Bash"
date: 2026-06-14 11:00:00 +0000
categories: [Tips]
tags: [bash, productivity, scripting]
author: Anjaney
---

In this post we will learn how to use loops in bash safely.

Use `find -print0` with null-delimited reads to avoid filename parsing errors.

```bash
find data -name "*.fastq.gz" -print0 | while IFS= read -r -d '' file; do
  echo "Processing $file"
done
```
Then we can process the file properly.
