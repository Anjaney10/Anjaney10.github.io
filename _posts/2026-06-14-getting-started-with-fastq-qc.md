---
layout: post
title: "Getting Started with FASTQ Quality Control"
date: 2026-06-14 09:00:00 +0000
categories: [Tutorials]
tags: [bioinformatics, fastq, qc, bash]
author: Anjaney
---

This tutorial walks through basic FASTQ quality checks using **FastQC**.

```bash
fastqc sample_R1.fastq.gz sample_R2.fastq.gz -o qc_reports
```

Review per-base quality and adapter content before downstream analysis.
