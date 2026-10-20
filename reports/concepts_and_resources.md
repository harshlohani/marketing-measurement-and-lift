# Concepts & Resources

A running glossary of concept deep-dives for this project — the "why does this
matter" explanations, kept here so they don't get lost once a chat scrolls away.
Each entry: what it is, why it matters for *this* project, how to check/fix it,
and where to read more.

---

## Core Learning Resources

Referenced throughout the entries below — listed once here instead of
repeated per-entry.

**Foundational (causal inference basics):**
- **Brady Neal — *Introduction to Causal Inference*** (free book + video
  course). Strong ML-oriented foundation; covers balance, confounding,
  potential outcomes rigorously but accessibly.
- **Matheus Facure — *Causal Inference for the Brave and True*** (free,
  Python-code-based). Good fit for this project specifically since it pairs
  concepts directly with runnable Python — matches a hands-on learning style.
- **Scott Cunningham — *Causal Inference: The Mixtape*** (free online book).
  More econometrics-flavored, good for building intuition with real examples.

**Applied / production (useful once you reach Phase 5+):**
- **DoWhy** and **EconML** (Microsoft) and **CausalML** (Uber) — production
  causal ML libraries, useful to browse for how causal inference is actually
  deployed in industry systems.

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

*(Resources: see Core Learning Resources above — this distinction is Chapter 1
material in all three foundational texts.)*

---cac

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
   industry term; production libraries exist (see Core Learning Resources —
   DoWhy/EconML, CausalML). Causal representation learning / invariant risk
   minimization is active research into *why* deep models generalize badly
   out-of-distribution (they latch onto spurious correlations instead of
   invariant/causal structure) — causal thinking is treated as a fix for a
   core weakness of deep learning, not a competing discipline.

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

---

## Unit of Randomization vs. Unit of Analysis (Duplicates Worked Example)

### What it means

The **unit of randomization** is whatever was actually assigned to
treatment/control (here, probably a customer). The **unit of analysis** is
whatever you're averaging over when you compute a rate or fit a model (here,
possibly a row/impression, if each customer can generate multiple rows).
When these two don't match, two distinct problems appear — neither of which
is about "the data being dirty":

1. **Reweighting bias.** If customers who get more impressions logged (more
   rows) also differ systematically on anything outcome-relevant, your
   row-level average silently becomes an impression-weighted average instead
   of a per-customer average — picking up a spurious correlation that has
   nothing to do with the treatment itself.
2. **Non-independence inflates false confidence.** Standard significance
   tests (z-test, chi-square, OLS standard errors) assume every row is an
   independent observation. If several rows actually come from one customer,
   the effective sample size is smaller than the row count suggests — p-values
   come out smaller (more "significant") than they should. This compounds the
   already-flagged problem of p-values being too easily triggered at 14M rows
   in the first place.

This matters specifically because duplicates are a **balance risk**, the same
way missing values are: if rows aren't evenly spread across treatment and
control, dropping or keeping them non-neutrally shifts your treatment ratio
and biases downstream estimates, silently.

### Worked example from this project

- `df.duplicated().sum()` → 1,259,545 exact-duplicate rows (~9% of 13,979,592).
- Treatment mean on full data: **0.850**. After `drop_duplicates()`: **0.839**.
  Treatment mean *within the duplicate rows only*: **0.961** — duplicates are
  heavily concentrated in the treated group, far more than the overall 85%
  treatment ratio. (Full numbers and the notebook check: see
  `01_data_understanding.ipynb`.)
- **No customer ID column exists** in this dataset, which is what makes this
  hard to resolve directly rather than just inconvenient.

**Checked Criteo's own documentation to settle what a row represents:**
both the official Criteo AI Lab page and the academic benchmark paper state
**"one row per user"** — verified directly from both primary sources (v1 was
25.3M rows, matching the stale-looking comment in the notebook; v2 is 14M
rows, matching this project's 13,979,592-row unbiased v2.1 subsample).

**But independent practitioner analysis complicates the official answer.**
Multiple independent write-ups of this exact dataset report that of the
13.98M rows, only **~9.17M distinct covariate combinations exist when
ignoring `f2`** — the other 11 features behave like a "proxy user ID," with
`f2` the only thing varying within a repeated group (consistent with what
this project noticed in its own first `head(10)` output). The sharpest
finding: **~3.4% of these proxy-ID clusters contain rows from *both*
treatment=1 and treatment=0** — a real single customer cannot be in both
arms of a between-subjects experiment, so even this richer reconstruction
isn't a clean customer ID. Some apparent "repeated customer" rows are
actually coincidental collisions between different customers.

*Confidence note: "one row per user" is directly verified from two primary
sources. The "9.17M distinct vectors / 3.4% cross-arm" claim is corroborated
across multiple independent secondary sources, but its original primary
source wasn't directly confirmed — treat it as well-supported, not certain.*

**Conclusion:** there is no way to cleanly separate "genuine repeat customer"
from "coincidental feature-bucket collision" from this data alone. This is a
real limitation of the dataset, not a gap in the analysis.

### Practical approach going forward

Don't resolve this at the understanding stage by picking an option. When
computing the first real ATE (Session 6 — treatment vs. control conversion
rate), run it **both with and without duplicates removed** as a robustness
check:
- If conclusions don't change, it's low-stakes — note it and move on.
- If they do change meaningfully, that's a dedicated follow-up (cluster-robust
  standard errors).

Either way, state "no customer ID; duplicate-row structure partially
unresolvable" explicitly as a limitation in the Phase 8 write-up.

---

## Feature Distributions — Why Look, Not Just Summarize

### What it means

A mean and standard deviation can hide completely different underlying
shapes — two features with identical mean/std can look nothing alike
plotted (a clean bell curve vs. two separate clusters vs. a tight group plus
a few extreme outliers dragging the mean). You can't tell which from numbers
alone.

### What a distribution check actually catches

1. **Skew and long tails** — a few extreme values can dominate a mean,
   making "the average customer" describe almost nobody. Already visible in
   this project's `describe()` output: `f6` ranges -31.4 to 0.29, `f9` has a
   max of 75.3 against a 25th percentile of 13.19.
2. **Low cardinality / discreteness** — a "continuous" feature with only a
   handful of distinct values is likely a bucketed/hashed categorical, not a
   true measurement. Directly extends the duplicates/proxy-ID investigation
   above.
3. **Multi-modality** — separate humps usually mean a mixture of genuinely
   different subpopulations lumped into one column.
4. **Sentinel values disguised as real numbers** — the sharpest trap. `isna()`
   only catches actual nulls; a dataset can encode "missing" as a real,
   valid-looking number instead (`-1`, `0`, `999`). A histogram shows this as
   a suspicious spike disconnected from the rest of the distribution —
   something a null-check will never surface.

### Why it matters more here than in a generic ML project

- **Protects upcoming balance checks (Session 5).** A mean-based SMD
  comparison on a skewed feature can be misleading either way — outliers in
  one group can fake an imbalance, or mask a real one. Knowing the shape
  first tells you whether to trust a raw mean comparison or need something
  more robust (median, log-transform).
- **Feeds Phase 4's heterogeneity analysis.** Naive quartile binning on a
  skewed feature can bury genuine treatment-effect heterogeneity in one
  overstuffed bucket.
- **More evidence for the proxy-ID question.** A feature with very few
  distinct values strengthens the "these are hashed/bucketed, not true
  continuous measurements" hypothesis already flagged as an open limitation.

### Worked example: `f0`'s real histogram confirms the bucketing hypothesis

Plotting `f0` turned the "proxy ID" hypothesis from inference into direct
evidence. `df['f0'].describe()` showed the **25th percentile exactly equal to
the minimum** (12.616) — a pattern a genuinely continuous measurement almost
never produces, since it means at least a quarter of all 13,979,592 rows hold
the exact same value. The histogram confirmed it at scale: the first bar
reaches ~3.9M rows, **~28% of the entire dataset**, matching the percentile
check almost exactly (two independent methods agreeing — the same
cross-validation habit as checking `value_counts(normalize=True)` against
`.mean()` earlier in this project).

The rest of the shape isn't a bell curve, simple skew, or clean bimodal
either: a flat, step-like low plateau from ~13.5–21, then a secondary rise
near the top of the range (22–27). Mean (19.62) and std (5.38) describe
essentially none of this — the single most common value isn't anywhere near
the mean, which is the real-data version of the "same mean/std, different
shape" lesson, no longer hypothetical.

This also closes the loop on the very first `head(10)` output from Session 1,
where `f0 = 12.616365` for all 10 rows shown — not a coincidence of early
rows, but the dominant value across over a quarter of the full dataset.

**Next diagnostic step, to confirm directly rather than infer from shape:**
```python
df['f0'].nunique()
df['f0'].value_counts().head(20)
```

---

## ECDF (Empirical Cumulative Distribution Function)

### What it means

For a value `x`, the ECDF answers: *"what fraction of rows have a value ≤ x?"*
Plotted, x-axis = feature value, y-axis = 0 to 1 cumulative share. It is built
directly from the sorted data: no bins, no bandwidth, no smoothing choices.
Every row contributes exactly one step.

### Why it beats histograms/KDE for comparing two groups

- **No binning or bandwidth artifacts.** A histogram's shape changes with bin
  width; a KDE's changes with bandwidth. The ECDF has no tuning knob, so a
  difference you see between groups is a difference in the data.
- **Spikes become vertical jumps.** A point mass (like `f0`'s ~28% at 12.616)
  shows as a vertical jump of height ~0.28 at that exact x. A KDE would blur
  it into a bump; a histogram hides it inside a bar. The jump height *is* the
  share of rows with that value, readable straight off the y-axis.
- **Sample-size independent.** y is a proportion, so a group with 85% of rows
  and a group with 15% are directly comparable. Raw-count histograms are not
  (the treated bars would dwarf the control bars).
- **Quantiles are read off directly.** Median = where the curve crosses 0.5.
  Comparing two groups = comparing their curves at every quantile at once,
  not just the mean.
- **Tails are visible.** Skewed features (`f6`, `f9`) show their long tail as
  a slow approach to 1.0 instead of invisible sliver bars.

### Why it matters specifically in this project

1. **It is a balance check (Session 5 preview).** Randomization implies the
   feature distribution is the same in treatment and control. Overlaid ECDFs
   test that on the *whole distribution*, while an SMD only compares means
   (scaled by std) and can miss a difference in shape or tails. Curves lying
   on top of each other = balanced; a consistent gap = investigate.
2. **The treatment/control split is 85/15.** That imbalance in group size makes
   count-based plots misleading; ECDFs are immune because they are proportions.
3. **Bucketed features.** Since `f0` and probably others are discrete/hashed,
   KDE comparisons are unreliable; ECDF handles discrete data exactly.
4. **No p-value trap.** At 14M rows a KS test (the max vertical gap between two
   ECDFs) will flag trivial differences as "significant". The *plot* shows
   whether a gap is visually large enough to matter, and the KS statistic
   itself (the gap size, not its p-value) is a usable effect size.

### How to read the plot

- Curves overlap → same distribution.
- One curve consistently above the other → that group has systematically
  smaller values (more of its mass sits to the left).
- Curves cross → groups differ in spread/shape, not just location.
- Max vertical gap = KS statistic; rule of thumb, gaps under ~0.01 are
  practically negligible.

### Practical notes

- Use a sample (`df.sample(500_000, random_state=0)`) for plotting speed; the
  ECDF of a 500k random sample is extremely close to the full one.
- Put `hue='treatment'` so both groups share one axis.
- Plot one feature per figure, or a grid of subplots with an explicit `ax=`
  (same `plt.figure()` vs `plt.Figure()` and axis pitfalls as in
  `useful_functions_reference.md`).
- Seaborn: `sns.ecdfplot`. Matplotlib-only: sort values, y = `np.arange(1, n+1)/n`.

*(Resources: Facure's *Causal Inference for the Brave and True* and Brady
Neal's book cover balance checks; the ECDF/KS idea itself is standard
nonparametric statistics.)*

---

## Histogram Bin Width and KDE Bandwidth

### What it means

Both control how much the data is smoothed before you look at it. Histogram
bin width and KDE bandwidth `h` play the same role: wide = coarse and stable,
narrow = detailed and noisy.

### The tradeoff (bias vs. variance)

- **Too wide:** stable but hides real structure (a KDE with large `h` melts
  `f0`'s 28% spike into a soft bump and puts density at values below the
  smallest real value).
- **Too narrow:** matches *this sample* closely but not the process behind it.
  Another random sample of the same feature gives a different jagged picture
  (overfitting). Smaller is not "better fit".
- **Limit, width -> 0:** a histogram cannot exist at exactly 0, but in the
  limit it becomes isolated needles, one per distinct observed value. Without
  ties all needles have equal height (1 row each), so the heights that carry
  the shape collapse. With ties (like `f0`) the needle heights equal
  `value_counts()`. That is just the raw data restated, not a shape.
- **Limit, 1 bin:** a single block holding every row. No shape either.
- A histogram is informative only when bars hold *many rows each*. A density
  estimate converges to the truth only if the width shrinks more slowly than
  the sample grows.
- The ECDF is the infinite-bin histogram accumulated left to right: each
  needle becomes a tiny step, so no information is lost and no width needs
  choosing. This is why it is the safe tool when you cannot trust a bin or
  bandwidth choice.

### Finding the sweet spot

Standard formulas pick a bin count/width from sample size `n` and data spread:

- **Sturges' rule:** best for roughly normal data.
  `bins = 1 + log2(n)`
- **Freedman-Diaconis rule:** better when there are outliers, because it uses
  the interquartile range (IQR) instead of the standard deviation.
  `bin width = 2 * IQR / n^(1/3)`

In NumPy/matplotlib: `bins='sturges'` or `bins='fd'` (or `'auto'`, which takes
the better of the two); `np.histogram_bin_edges(x, bins='fd')` returns the
edges so you can inspect them. KDE has the analogous rules of thumb: Scott's
and Silverman's bandwidth (the defaults in `sns.kdeplot` and pandas'
`plot(kind='kde')`).

### Caveats specific to this project

- **Sturges is weak at this scale.** With n = 13,979,592 it gives
  `1 + log2(n) ~ 24.7`, so about 25 bins. It only grows logarithmically and
  was designed for small, normal-ish samples.
- **Freedman-Diaconis scales better.** `n^(1/3) ~ 241`, so the width is about
  `IQR / 120`, i.e. many fine bins. Compute each feature's IQR from
  `describe()` to see what it suggests.
- **Neither rule knows about point masses.** Both assume a smooth
  distribution. `f0`'s ~28% spike at 12.616 (25th percentile = minimum) is
  real repeated values, so it stays one tall bar at any bin count, and the
  rules cannot flag it. Treat the rule output as a starting point and confirm
  with `nunique()` / `value_counts()` for suspect features.
- **Smoothing hides what the check is for.** In distribution checks the goal
  is to *find* spikes, sentinels and bucketing, so lean toward less smoothing
  for large n, and cross-check with ECDFs rather than trusting any single
  setting.

### Interactive explainers

- KDE and ECDF Lab: https://claude.ai/artifact/49mkSnxMFp8XBRLt1CAEe3
- Bin Width Lab: https://claude.ai/artifact/TC9vu3qHzUkV9ERL4RVdwb
