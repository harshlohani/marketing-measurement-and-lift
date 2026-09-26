# Incremental Marketing & Customer Response Modeling

## Project Overview

This project focuses on **incremental marketing measurement and uplift modeling**.

The central question is not simply:

> **Which customers are most likely to buy?**

Instead, we want to answer the more important marketing question:

> **Which customers are more likely to buy because we showed them an advertisement?**

This distinction is the foundation of the project.

A customer may have a very high probability of purchasing even if we do nothing. Spending advertising money on that customer may therefore generate little or no incremental value.

Another customer may have a low baseline probability of purchasing, but advertising could substantially increase their probability of conversion. That customer may be much more valuable to target.

The project will therefore move from conventional predictive modeling toward **causal inference, heterogeneous treatment effects, and uplift modeling**.

---

## The Business Problem

Imagine a company has millions of customers and a limited advertising budget.

For every customer, there are conceptually two possible outcomes:

- What would happen if the customer **received the advertisement**
- What would happen if the customer **did not receive the advertisement**

The difference between these two outcomes represents the customer's **incremental treatment effect**.

Conceptually:

\[
\text{Incremental Effect}
=
P(\text{Conversion} \mid \text{Ad})
-
P(\text{Conversion} \mid \text{No Ad})
\]

In reality, we cannot observe both outcomes for the same individual at the same time. This is the fundamental **causal inference / potential outcomes problem**.

The project will investigate how randomized treatment data can help us estimate these effects and use them to make better customer-targeting decisions.

---

## Why Normal Predictive Modeling Is Not Enough

A conventional conversion model might answer:

> "Who is most likely to purchase?"

That can be useful, but it does not necessarily tell us who should receive advertising.

Consider four hypothetical customers:

| Customer | Conversion without ad | Conversion with ad | Incremental effect |
|---|---:|---:|---:|
| A | 90% | 92% | +2% |
| B | 5% | 30% | **+25%** |
| C | 80% | 80% | 0% |
| D | 30% | 10% | **−20%** |

A normal predictive model may prioritize Customer A because the customer has a very high probability of converting.

An incremental marketing model would recognize that Customer B receives substantially more benefit from the advertising treatment.

Customer C may convert anyway, meaning advertising provides little incremental benefit.

Customer D could even be negatively affected by the treatment.

This is why the project focuses on **incrementality rather than prediction alone**.

---

## Why We Chose the Criteo Uplift Prediction Dataset

The **Criteo Uplift Prediction Dataset** was specifically chosen because it is well suited to this problem.

Criteo created the dataset from **randomized controlled advertising experiments / incrementality tests**. Users were assigned to treatment and control groups, allowing us to study the difference between advertising exposure and the absence of advertising.

This is particularly valuable for our project because we want to learn:

1. How randomized treatment data works
2. How treatment and control groups should be compared
3. How to estimate treatment effects
4. How treatment effects can vary across customers
5. How to build uplift models
6. How to turn model output into a customer-targeting policy
7. How to connect statistical results to business value

In other words, the dataset allows us to work on a problem that is much closer to **real marketing decision-making** than simply predicting an outcome.

---

## Why Criteo Is Particularly Useful for This Project

### 1. It contains treatment information

The dataset distinguishes customers who received the advertising treatment from those in the control group.

This gives us the fundamental structure needed for treatment-effect analysis.

### 2. It contains conversion outcomes

We can study whether customers eventually converted and compare outcomes between treatment and control populations.

### 3. It contains customer-level features

The dataset contains anonymized customer features (`f0`–`f11`).

These features allow us to investigate whether treatment effects differ across different types of customers.

### 4. It is large enough to make the problem realistic

The Criteo dataset contains approximately **14 million observations**.

That gives us an opportunity to work with a dataset much closer to the scale encountered in modern marketing analytics rather than a tiny educational dataset.

### 5. It naturally leads from basic statistics to advanced data science

The same dataset supports the entire progression of this project:

**EDA → statistical testing → predictive modeling → causal inference → heterogeneous treatment effects → uplift modeling → targeting policy → business impact**

This makes it particularly suitable for a portfolio project because the work does not depend on building several unrelated models.

---

# What We Are Going to Build

The project will be developed progressively.

## Phase 1 — Understand the Dataset and Business Problem

We will first understand:

- What each variable represents
- Treatment and control groups
- Conversion and visit outcomes
- Exposure information
- Feature distributions
- Missing values and data quality
- Treatment/control balance

The goal is to understand the data before modeling.

---

## Phase 2 — Exploratory Data Analysis and Statistical Treatment Effects

We will investigate questions such as:

- What is the overall conversion rate?
- How does conversion differ between treatment and control?
- Is there evidence of an overall treatment effect?
- Are treatment and control groups balanced?
- Do treatment effects appear to differ across customer characteristics?

This establishes the statistical foundation for the later modeling work.

---

## Phase 3 — Baseline Predictive Models

Before building uplift models, we will establish conventional baselines.

Potential models include:

- Logistic regression
- Decision trees
- Random forests
- Gradient boosting models

The purpose is not simply to maximize predictive accuracy.

It is to understand the difference between:

**predicting conversion**

and

**predicting incremental response to treatment.**

---

## Phase 4 — Causal Inference and Heterogeneous Treatment Effects

We will introduce the causal inference framework.

Topics will include:

- Potential outcomes
- Average Treatment Effect (ATE)
- Conditional / heterogeneous treatment effects
- Treatment-control comparisons
- Confounding and why randomization matters
- Individual treatment-effect estimation
- Model-based heterogeneous effects

This is the conceptual bridge between conventional machine learning and uplift modeling.

---

## Phase 5 — Uplift Modeling

We will build models designed specifically to estimate incremental response.

The goal is to rank customers according to their expected incremental benefit from treatment.

We will investigate appropriate uplift modeling approaches and compare their performance.

---

## Phase 6 — Customer Targeting Policy

The model should ultimately lead to a business decision.

Instead of simply producing:

> "Customer X has a 73% probability of conversion."

we want something closer to:

> "Customer X is estimated to have a high incremental response to advertising."

Customers can then be segmented according to estimated incremental value.

For example:

- High incremental response
- Moderate incremental response
- Low incremental response
- No incremental response
- Potentially negative response

The exact segmentation will be determined from the analysis rather than assumed in advance.

---

## Phase 7 — Incremental Profit and Business Impact

A model is much more useful when it can be connected to economics.

We will introduce a hypothetical marketing cost / revenue framework and ask:

> **If we have a limited marketing budget, which targeting strategy generates the greatest incremental business value?**

We can compare strategies such as:

- Target everyone
- Target customers with high predicted conversion probability
- Target customers with high estimated uplift

The objective is to demonstrate why incremental modeling can change the marketing decision.

---

## Phase 8 — Final Portfolio and Interview Preparation

The final project will include:

- Cleaned and documented analysis
- Reproducible Python code
- EDA
- Statistical analysis
- Baseline models
- Causal analysis
- Uplift modeling
- Model evaluation
- Customer targeting strategy
- Business impact analysis
- GitHub README
- Visualizations
- Key findings
- Limitations
- Interview discussion points

The final objective is to be able to explain the entire project clearly in a **Data Scientist / Data Modeler interview**, including why each modeling decision was made.

---

# Why This Project Fits My Career Goal

My goal is to transition from **Data Analyst → Data Scientist / Modeler**.

I already have experience working with:

- Data preparation
- SQL
- R
- Tableau
- Marketing analytics
- Market Mix Modeling (MMM)
- Multi-Touch Attribution (MTA)

This project is intended to extend that foundation into:

- Python-based machine learning
- Statistical modeling
- Causal inference
- Treatment-effect estimation
- Uplift modeling
- Model evaluation
- Business optimization

The project therefore connects my existing marketing analytics experience with the data science skills I want to develop.

---

# Project Learning Philosophy

This project is deliberately being built progressively.

I do **not** want to copy a finished notebook or immediately jump into sophisticated libraries.

For each stage, the process will be:

1. Understand the business question
2. Learn the statistical / machine learning concept
3. Implement it in Python
4. Interpret the result
5. Connect the result to the business problem
6. Document what was learned
7. Move to the next stage

The goal is not merely to produce a GitHub repository.

The goal is to develop enough understanding that I can **defend the methodology, code, assumptions, results, and business implications in an interview.**

---

# Scope Discipline

This is intended to be the primary data science portfolio project during the current learning period.

New project ideas should **not** replace this project simply because they seem interesting.

If another project idea comes up, it should be placed in a **parking lot** and revisited only after this project is substantially completed.

The priority is:

> **Finish one strong project deeply rather than starting many projects superficially.**

---

# Planned Project Timeline

Target duration: approximately **5 weeks**.

### Week 1
Dataset understanding, business problem, EDA, treatment/control analysis.

### Week 2
Baseline models, logistic regression, feature engineering, and model evaluation.

### Week 3
Causal inference and heterogeneous treatment effects.

### Week 4
Uplift modeling and customer-targeting policy.

### Week 5
Incremental profit/business impact, GitHub documentation, project polish, and interview preparation.

A two-week MVP may be completed earlier, but the full project is intentionally given more time so that the concepts are understood rather than simply implemented.

---

# The Central Question

Everything in this project ultimately comes back to one question:

> **Instead of asking who is likely to buy, can we identify who is likely to buy because of our marketing intervention?**

That is the reason for choosing this problem, this dataset, and this modeling approach.
