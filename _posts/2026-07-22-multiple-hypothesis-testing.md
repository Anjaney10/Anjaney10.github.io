---
layout: post
title: "Mathematical reasoning behind multiple hypothesis testing and correction"
date: 2026-07-23 09:00:00 +0000
math: true
categories: [Tutorials]
tags: [biostats, deseq, maths]
author: Anjaney
---

In this blog, we will be talking about some key concepts related to differential expression analysis, especially when it comes to filtering which genes are significant. Usually, people calculate the p-value (one of the most misunderstood ideas in stats) and call it a day. But what they don't realise is that the actual signal is still hiding behind the layers of false positives. So, how do we identify the "truly" differentially expressed genes? 

Here's a condensed walk-through of this problem and some well-known solutions:

## Type I Error Inflation

Single hypothesis test:
Set significance level $\alpha = P(\text{Reject } H_0 \mid H_0 \text{ true}) = P(\text{Type I error})$.

Testing $m$ independent hypotheses simultaneously ($m \ge 10^4$ in DEG analysis):


$$P(\text{At least one false positive}) = 1 - P(\text{No false positives}) = 1 - (1 - \alpha)^m$$

For $\alpha = 0.05$ and $m = 20{,}000$:


$$1 - (1 - 0.05)^{20000} \approx 1$$

Without correction, the false positive rate approaches $100\%$.

---

## Measuring Error

Testing $m$ total hypotheses ($m_0$ true $H_0$, $m - m_0$ false $H_0$):

| Decision / Truth | $H_0$ True | $H_0$ False | Total |
| --- | --- | --- | --- |
| **Declared Significant (Reject $H_0$)** | $V$ (Type I Error / False Positive) | $S$ (True Positive) | $R$ |
| **Declared Non-Significant (Fail to Reject $H_0$)** | $U$ (True Negative) | $T$ (Type II Error / False Negative) | $m - R$ |
| **Total** | $m_0$ | $m - m_0$ | $m$ |

$V, S, U, T, R$ are random variables. $m_0, m$ fixed.

### Family-Wise Error Rate (FWER)

Probability of committing at least one Type I error across all tests:


$$FWER = P(V \ge 1)$$

### False Discovery Rate (FDR)

Expected proportion of false positives among all rejected null hypotheses:


$$FDR = E[Q] \quad \text{where} \quad Q = \begin{cases} \frac{V}{R} & \text{if } R > 0 \\\\ 0 & \text{if } R = 0 \end{cases}$$

---

## Multiple Testing Adjustment Methods

### A. Bonferroni Correction (FWER Control)

Controls $FWER \le \alpha$. Reject $H_{0,i}$ if $p_i \le \frac{\alpha}{m}$.

#### Mathematical Proof:

Let $I_0 \subset \{1, \dots, m\}$ be set of true null hypotheses ($\vert{}I_0\vert{} = m_0$).
Under $H_0$, $P_i \sim \text{Uniform}(0, 1)$, so $P(P_i \le c) = c$.

$$FWER = P(V \ge 1) = P\left( \bigcup_{i \in I_0} \left\\{ P_i \le \frac{\alpha}{m} \right\\} \right)$$

By Boole's Inequality (Union Bound):


$$P\left( \bigcup_{i \in I_0} \left\\{ P_i \le \frac{\alpha}{m} \right\\} \right) \le \sum_{i \in I_0} P\left( P_i \le \frac{\alpha}{m} \right) = m_0 \cdot \frac{\alpha}{m} \le m \cdot \frac{\alpha}{m} = \alpha$$

*Limitation:* Overly conservative when $m$ large. High Type II error (low statistical power).

---

### B. Benjamini-Hochberg (BH) Procedure (FDR Control)

Controls $FDR \le q^*$ target threshold. Standard choice for DEG analysis.

#### Algorithm:

1. Sort raw p-values in ascending order: $p_{(1)} \le p_{(2)} \le \dots \le p_{(m)}$.
2. Find maximum index $k$:

$$k = \max \left\\{ i \in \{1, \dots, m\} : p_{(i)} \le \frac{i}{m} q^* \right\\}$$


3. Reject $H_{(1)}, H_{(2)}, \dots, H_{(k)}$. If no such index $i$ exists, reject none.

#### Adjusted P-values (q-values):

To enforce monotonicity, calculate q-value for order statistic $i$:


$$q_{(i)} = \min_{j \ge i} \left( \min \left( 1, \frac{m \cdot p_{(j)}}{j} \right) \right)$$

---

## Full Workflow for DEG Analysis

In RNA-seq differential expression analysis, $m \approx 20{,}000 - 60{,}000$ genes tested simultaneously.

Raw Reads -> Count Matrix -> Distribution Model -> Hypothesis Test (p-values) -> Independent Filter -> BH Correction (q-values) -> Threshold Filtering

### Step 1: Statistical Modeling per Gene $i$

Count data exhibits overdispersion ($\text{Var}(X) > \text{E}[X]$). Model gene expression count $Y_{ij}$ using Negative Binomial distribution:


$$Y_{ij} \sim \text{NB}(\mu_{ij}, \alpha_i)$$

Fit Generalized Linear Model (GLM) for log-fold change $\beta_i$:


$$\log_2(\mu_{ij}) = x_j^T \boldsymbol{\beta}_i$$

* Null hypothesis $H_{0,i}: \beta_{i, \text{condition}} = 0$ (no differential expression).
* Calculate raw p-value $p_i$ via Wald Test or Likelihood Ratio Test (LRT).

### Step 2: Independent Filtering

Remove low-count genes before correction.

* *Math rationale:* Low counts offer insufficient power to beat $p_i \le \frac{i}{m}q^*$. Removing them reduces $m$, lowering penalty on well-expressed genes without affecting Type I error control.

### Step 3: FDR Adjustment

Apply Benjamini-Hochberg transformation to raw p-values $p_1, \dots, p_m$ to generate q-values $q_1, \dots, q_m$.

### Step 4: Final Selection Threshold

Gene $i$ classified as a Differentially Expressed Gene (DEG) iff:

1. $q_i \le q^*$ (typically $0.05$)
2. $\vert{}\text{log}_2 \text{FC}_i\vert{} \ge \text{threshold}$ (typically $\ge 1.0$, corresponding to 2-fold change)
