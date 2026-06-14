---
title: "What Is Differential Expression Analysis?"
date: 2026-06-14 10:00:00 +0000
categories: [Explainers]
tags: [rna-seq, statistics, concepts, r]
author: Anjaney
---

Differential expression analysis identifies genes with significant expression changes between conditions.

```r
results <- DESeq2::results(dds)
head(results[order(results$padj), ])
```

Focus on both statistical significance and effect size.
