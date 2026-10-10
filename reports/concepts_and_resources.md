# Concepts & Resources

A running glossary of concept deep-dives for this project — the "why does this
matter" explanations, kept here so they don't get lost once a chat scrolls away.
Each entry: what it is, why it matters for *this* project, how to check/fix it,
and where to read more.

---

## Balance (covariate balance between treatment and control)

### What it means

"Balance" asks: do the treatment group and control group look statistically
similar on every feature you can measure (`f0`–`f11` here), aside from the
treatment itself?

In a properly randomized experiment, balance should hold **on average** — every
feature's distribution should be roughly the same in both groups, because
`treatment` was assigned by a coin flip, independent of any customer
characteristic.

### Why it matters

The entire reason a simple difference in conversion rates (treatment vs.
control) is allowed to be interpreted as a causal effect is that randomization
makes the two groups comparable. If the groups are *not* balanced on some
feature that also affects conversion, then part of the observed difference in
outcomes could be due to that feature, not the treatment — the comparison
becomes confounded, exactly like a non-randomized observational study.

Balance can break in three distinct ways, worth telling apart:

1. **Chance imbalance** — even with true randomization, any single random split
   can look imbalanced on some feature just from sampling variability. Less
   likely at 14M rows, but not impossible for a feature with high variance.
2. **A broken randomization process** — if balance is badly off on several
   features, that's a sign the treatment assignment wasn't actually random
   (e.g. a bug in how Criteo's experiment was run, or in how the data was
   exported). This would be a serious, project-changing finding if found.
3. **Imbalance introduced by your own preprocessing** — e.g. dropping rows with
   missing values non-randomly (see the missing-values note in
   `01_data_understanding.ipynb`) can turn a balanced dataset into an unbalanced
   one after the fact.

### How to check it

- **Don't rely on raw p-values / t-tests at this scale.** With 14M rows, even a
  trivially small, practically meaningless difference between groups will
  often come back "statistically significant" — the test has enormous power to
  detect tiny differences. A low p-value here doesn't tell you the imbalance is
  *large enough to matter*.
- **Use Standardized Mean Difference (SMD) instead**, which is scale-free and
  not inflated by sample size:

  ```
  SMD = (mean_treatment - mean_control) / pooled_std
  ```

  Rule of thumb commonly used in the causal inference literature:
  - `|SMD| < 0.1` → good balance
  - `0.1–0.25` → moderate, worth a closer look
  - `> 0.25` → meaningful imbalance, investigate

- **Visualize it** with a "Love plot" — one row per feature, SMD plotted as a
  point, with reference lines at ±0.1. This is the standard "Table 1" style
  check in any RCT/causal paper — worth building as a habit here (maps to
  Session 5 in `criteo_uplift_project_working_sessions.md`).

### How to "fix" imbalance if you find it

Most relevant to observational (non-randomized) data, but still useful if a
randomized dataset shows residual imbalance:

- **Regression adjustment** — include the unbalanced feature as a covariate in
  the model used to estimate the effect. This reduces variance and corrects for
  small residual imbalance even when randomization is basically sound.
- **Stratification** — split the data into bins of the unbalanced feature,
  estimate the effect within each bin, then combine (e.g. a weighted average).
- **Inverse probability weighting (IPW)** — reweight observations so the
  weighted feature distributions match across groups.
- **Matching** (e.g. propensity score matching) — mostly a tool for
  observational studies; usually unnecessary when you have genuine
  randomization, but sometimes used as an extra robustness check.

For this project specifically: since Criteo's data comes from a genuine
randomized incrementality test, the expectation is good balance by default.
The point of Session 5 is to **verify** that rather than assume it — and if
something looks off, to figure out whether it's (1) harmless chance noise,
(2) a real problem with the experiment, or (3) something your own
preprocessing introduced.

### Resources

- **Brady Neal — *Introduction to Causal Inference*** (free book + video
  course). Strong ML-oriented foundation; covers balance, confounding,
  potential outcomes rigorously but accessibly.
- **Matheus Facure — *Causal Inference for the Brave and True*** (free,
  Python-code-based). Good fit for this project specifically since it pairs
  concepts directly with runnable Python — matches a hands-on learning style.
- **Scott Cunningham — *Causal Inference: The Mixtape*** (free online book).
  More econometrics-flavored, good for building intuition with real examples.

---

## Causal Modeling vs. Predictive Modeling

### What it means

Predictive modeling answers: *"given X, what is Y?"* — a question about
**observation**. It estimates `P(Y | X)`.

Causal modeling answers: *"given I **change** X, what happens to Y?"* — a
question about **intervention**. In Pearl's notation, it estimates
`P(Y | do(X))`.

These are not the same quantity, and treating them as interchangeable is the
most common mistake in applied ML.

### Why they diverge — the classic failure mode

Ice cream sales correlate with drowning deaths. A predictive model exploits
that correlation perfectly for prediction — and would be completely wrong if
acted on causally (banning ice cream does nothing to drownings). Both are
driven by a hidden common cause: hot weather. The correlation is real; the
causal story it implies is not. This is **confounding**.

### How it shows up in this exact project

Phase 3's baseline model (`conversion ~ f0...f11`, ignoring `treatment`) ranks
customers by their **baseline propensity to buy** — whatever features
correlate with "already intends to purchase." This is exactly Customer A from
the README: 90% baseline conversion, looks great to a predictive model, but
only +2% uplift from the ad. The predictive model isn't wrong at its own task
— it's answering a different question than "who should we advertise to?"

### Four concrete contrasts

| | Predictive | Causal / Uplift |
|---|---|---|
| Target | `P(Y\|X)` | `E[Y\|do(T=1)] − E[Y\|do(T=0)]` |
| Ground truth per row? | Yes — you observe `Y` | **No, never** — you can't observe the same customer both with and without treatment (the "fundamental problem of causal inference") |
| Good feature engineering | Any feature correlated with Y helps | Must distinguish **confounders** (common cause of T and Y — adjust for these) from **mediators** (T → feature → Y, lies on the causal path) and **colliders** (T and Y both cause the feature) — adjusting for mediators/colliders *introduces* bias rather than removing it |
| What fixes a wrong answer | More data, better features | Only a valid design (randomization) or a credible identification assumption — no amount of extra observational data fixes an unmeasured confounder |

The "no ground truth per row" row is the one with the most downstream
consequences: it's exactly why Phase 5 needs Qini/AUUC instead of
accuracy/AUC — you're evaluating against an aggregate pattern across many
customers, never a per-row label, because that label fundamentally cannot
exist for any single customer.

### Resources

- Same three as above (Brady Neal, Facure, Cunningham) — this distinction is
  Chapter 1 material in all three.

---

## Why Causal Inference Matters — and Whether AI Changes That

### What it means

Prediction answers "what is Y given X, as the world already is." Almost every
real decision is instead a causal question: "what happens to Y if we *change*
X?" The gap between those two questions is where costly mistakes live.

### Why AI doesn't make this obsolete — it sharpens it

1. **LLMs/foundation models are trained on correlational, observational data**
   (next-token prediction = `P(text | context)`, no `do()` anywhere). They can
   produce fluent causal-*sounding* language because they've seen humans write
   causal sentences — that's different from performing valid causal inference.
   Sounding causal ≠ being causally valid, and fluency makes the gap harder to
   notice.
2. **Automated decision systems create feedback loops.** A recommender/targeting
   system that only ever acts on its own past decisions contaminates its own
   training data — structurally the same problem Criteo's randomized design
   was built to solve. This is why ad tech specifically was an early adopter of
   causal inference, not an academic curiosity.
3. **Scale multiplies the cost of the mistake.** One bad human call wastes one
   decision; the same statistical error running inside an automated system
   making millions of decisions/day multiplies into large misallocated
   spend or harm, silently.
4. **It's now a named frontier inside AI research itself** — "causal AI" is an
   industry term; production libraries exist (Microsoft's DoWhy/EconML, Uber's
   CausalML). Causal representation learning / invariant risk minimization is
   active research into *why* deep models generalize badly out-of-distribution
   (they latch onto spurious correlations instead of invariant/causal
   structure) — causal thinking is treated as a fix for a core weakness of
   deep learning, not a competing discipline.

### Resources

- Same three (Brady Neal, Facure, Cunningham) for foundations.
- **DoWhy** and **EconML** (Microsoft) and **CausalML** (Uber) — production
  causal ML libraries, useful to browse for how causal inference is actually
  deployed in industry systems.
