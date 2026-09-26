# Working Sessions — Criteo Uplift Project

This breaks the 8 phases from `criteo_uplift_project_README.md` into concrete
~1.5–2 hour working sessions (matching the ~1.5–2 hrs/day pace from the chat
archive). Each session has one objective, the concept to understand first,
the task to implement yourself, and a stop point / deliverable.

Check sessions off as you go. Don't feel bound to exactly one session per day —
merge or split sessions based on how fast a concept clicks. If a session's
concept takes 3 days instead of 1, that's fine — see "Scope Discipline" and
the timeline note in the README.

Total: ~28 sessions, ~42–56 hours — consistent with the 5-week / 40–50 hour
estimate in the chat archive.

---

## Phase 1 — Understand the Dataset and Business Problem

- [ ] **Session 1 — Load and verify the data**
  Objective: confirm the dataset loads correctly and matches the documented
  shape (13,979,592 rows; f0–f11 + treatment, conversion, visit, exposure).
  Task: run your existing load cell, `df.head()`, `df.shape`. Fix the stale
  `# Rows: 25,309,483` comment once you see the real number.
  Stop when: you've pasted/recorded the real shape and head output.

- [ ] **Session 2 — Structural investigation**
  Concept: why checking dtypes/nulls/duplicates *before* modeling matters
  (silent bugs vs. loud errors).
  Task: `df.info()`, `df.isnull().sum()`, `df.duplicated().sum()`.
  Stop when: you've written a short markdown cell summarizing what you found.

- [ ] **Session 3 — Group distributions**
  Concept: what a "treatment ratio of .846" structurally means — this is a
  designed imbalance in the experiment, not a bug.
  Task: `value_counts(normalize=True)` on `treatment`, `visit`, `conversion`,
  `exposure`.
  Stop when: you've compared your numbers to the documented ones (.846
  treatment ratio, ~.041 visit rate, ~.002 conversion rate) and written 3–4
  sentences on whether they match.

---

## Phase 2 — EDA and Statistical Treatment Effects

- [ ] **Session 4 — Feature distributions**
  Task: histograms / summary stats (`describe()`) for `f0`–`f11`.
  Concept: why distribution shape (skew, outliers, scale) matters for models
  you'll build later.

- [ ] **Session 5 — Treatment vs. control balance check**
  Concept: covariate balance — why you check that treatment/control groups
  look statistically similar *before* trusting a randomized comparison.
  Task: compare mean of each feature between `treatment==1` and
  `treatment==0` groups.

- [ ] **Session 6 — Your first real treatment effect**
  Concept: under randomization, a simple difference in group means *is* an
  ATE estimate.
  Task: compute conversion rate for treatment vs. control, take the
  difference. This is your first Average Treatment Effect.

- [ ] **Session 7 — Is it statistically significant?**
  Concept: hypothesis testing — why a small percentage-point difference could
  be noise, even at 14M rows (or could be highly significant — that's the
  point of testing it).
  Task: two-proportion z-test or chi-square test on conversion rates.

- [ ] **Session 8 — Visit-level effect + write-up**
  Task: repeat Session 6–7 for `visit` instead of `conversion`.
  Stop when: you've written up your first "is there a treatment effect?"
  finding in a markdown cell or `reports/` note.

---

## Phase 3 — Baseline Predictive Models

- [ ] **Session 9 — First baseline model**
  Task: train/test split, logistic regression predicting `conversion` from
  `f0`–`f11` (ignore `treatment` for now — this is deliberately the "wrong"
  question, to contrast with uplift later).

- [ ] **Session 10 — Evaluate the baseline**
  Task: AUC, precision/recall, confusion matrix.
  Concept: "this model predicts conversion well" is *not* the same as "this
  tells us who to target with ads."

- [ ] **Session 11 — Tree-based baseline**
  Task: random forest and/or gradient boosting, compare to logistic
  regression.
  Concept: feature importance in this model is *predictive* importance, not
  *causal* importance — flag this explicitly, don't let it get confused later.

- [ ] **Session 12 — Reflect and connect back to the business problem**
  Task: write a short note connecting this baseline's limitations to the
  Customer A/B/C/D table in the README. Why would targeting by this model's
  output be a mistake for Customer A?

---

## Phase 4 — Causal Inference and Heterogeneous Treatment Effects

- [ ] **Session 13 — Potential outcomes framework (concept only, no code)**
  Task: write the potential-outcomes idea in your own words — the two
  outcomes per customer, why you never observe both, why that's the
  "fundamental problem of causal inference."

- [ ] **Session 14 — ATE with a confidence interval**
  Task: revisit Session 6's ATE estimate, add a confidence interval around it.

- [ ] **Session 15 — Confounding vs. randomization (concept only)**
  Task: construct or read through a toy example where an observational
  (non-randomized) comparison gives a misleading effect estimate. Explain why
  Criteo's randomized design avoids that problem.

- [ ] **Session 16 — Your first heterogeneous effect estimator: T-learner**
  Task: fit two separate models — one on treatment-only rows, one on
  control-only rows — and take the difference in predicted probabilities per
  customer. This is the simplest possible CATE (individual treatment effect)
  estimator.

- [ ] **Session 17 — Do effects vary by segment?**
  Task: bucket customers by a feature (e.g. quartiles of `f0`) and compare
  average T-learner CATE estimates across buckets. Is there evidence of
  heterogeneity?

---

## Phase 5 — Uplift Modeling

- [ ] **Session 18 — Explore an uplift library**
  Task: install and read docs for `scikit-uplift` or `causalml` (no modeling
  yet — just understand what they offer vs. your manual T-learner).

- [ ] **Session 19 — Fit a library uplift model**
  Task: fit an off-the-shelf uplift model (e.g. uplift tree, X-learner) and
  compare its structure to your manual T-learner.

- [ ] **Session 20 — Qini curve / AUUC (concept only)**
  Concept: why standard metrics (accuracy, AUC) don't work for uplift — you
  never observe ground-truth individual uplift, only aggregate curves.

- [ ] **Session 21 — Evaluate uplift models**
  Task: compute Qini/AUUC for your T-learner and library model, compare.

---

## Phase 6 — Customer Targeting Policy

- [ ] **Session 22 — Rank and segment customers**
  Task: rank customers by predicted uplift, define segments (e.g. high /
  moderate / low / negative incremental response).

- [ ] **Session 23 — Simulate targeting policies**
  Task: simulate three policies — target everyone, target top-k by predicted
  conversion probability, target top-k by predicted uplift. Compare expected
  outcomes.

---

## Phase 7 — Incremental Profit and Business Impact

- [ ] **Session 24 — Define a cost/revenue framework**
  Task: pick a hypothetical cost-per-ad and revenue-per-conversion, document
  your assumptions clearly (this is a portfolio project — the assumptions
  matter as much as the numbers).

- [ ] **Session 25 — Compute incremental profit per policy**
  Task: apply the cost/revenue framework to the three policies from Session
  23. Which one maximizes incremental profit under a fixed budget?

---

## Phase 8 — Final Portfolio and Interview Preparation

- [ ] **Session 26 — Consolidate the narrative**
  Task: clean up notebooks into a coherent story, update the top-level
  README with actual results (not just the plan).

- [ ] **Session 27 — Key findings and limitations**
  Task: write an explicit "what this project shows, and what it doesn't"
  section — limitations are a strength in an interview, not a weakness.

- [ ] **Session 28 — Interview prep**
  Task: write out (or say out loud) answers to: "why not just predict
  conversion?", "how would you know your uplift model is any good?", "what
  would you do differently with more time/data?"

---

## Notes

- Row-count discrepancy flagged in Session 1: notebook comment said
  25,309,483 rows; confirmed correct figure for the unbiased v2.1 dataset is
  **13,979,592 rows** — verify against actual `df.shape` output.
- If a phase's session count feels off once you're in it (causal inference
  in particular tends to take longer than planned), adjust rather than rush —
  see the timeline note in the README: target 5 weeks, minimum viable 2
  weeks, max ~6 weeks.
