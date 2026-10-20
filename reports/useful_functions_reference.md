# Useful Functions & Methods Reference

A running cheat sheet of pandas/sklearn functions worth remembering, with what
they do, when to reach for them, and an example tied to this project. This is
the practical/code-level companion to `concepts_and_resources.md` (which holds
the conceptual "why does this matter" deep-dives).

Format per entry: **what it does** → **when to use it here** → **example**.

---

## `value_counts(normalize=True)`

**What it does:** by default, `Series.value_counts()` returns raw counts per
unique value. `normalize=True` divides each count by the total, returning
proportions instead (they sum to 1).

**When to use it:** any time you want a *rate* rather than a raw count — e.g.
treatment ratio, visit rate, conversion rate. Works on any categorical column
with any number of distinct values, not just binary 0/1 columns.

**Example:**
```python
df['treatment'].value_counts(normalize=True)
# 1    0.85
# 0    0.15
```

**Related:** for a strictly binary 0/1 column, `.mean()` gives the same number
for the "1" category — `mean = (number of 1s) / total rows`, which is exactly
what `value_counts(normalize=True)` computes for that value. Good cross-check:
if these two ever disagree, something upstream is wrong.

---

## `pd.crosstab(..., normalize=...)`

**What it does:** builds a frequency table between two (or more) columns, with
an optional `normalize` argument: `'index'` (rows sum to 1), `'columns'`
(columns sum to 1), or `True`/`'all'` (whole table sums to 1).

**When to use it here:** comparing an outcome rate *between groups* in one
line — exactly the treatment-vs-control comparison this project is built on.

**Example:**
```python
pd.crosstab(df['treatment'], df['conversion'], normalize='index')
# gives conversion rate within treatment=1 and within treatment=0 separately
```

---

## `groupby(...).value_counts(normalize=True)`

**What it does:** same idea as `crosstab`, via groupby instead — proportions
of a column's values, computed separately within each group.

**When to use it here:** equivalent to the crosstab example above; a matter
of which reads more clearly for the situation (crosstab tends to be cleaner
for a simple two-column comparison, groupby scales better to more groups or
chaining with other groupby operations).

**Example:**
```python
df.groupby('treatment')['conversion'].value_counts(normalize=True)
```

---

## `groupby(...).mean()` — for balance checks

**What it does:** computes the mean of every numeric column, separately per
group.

**When to use it here:** checking covariate balance between treatment and
control (see "Balance" entry in `concepts_and_resources.md`) — a first,
simple look at whether `f0`–`f11` look similar across the two groups.

**Example:**
```python
df.groupby('treatment').mean()
```

Note: this is a *first pass* — the proper balance check uses Standardized
Mean Difference (SMD), not just eyeballing two columns of means. See
`concepts_and_resources.md` for why raw comparison isn't enough at 14M rows.

---

## `df.isna().sum()` / `df.duplicated().sum()`

**What they do:** `.isna()` flags nulls element-wise, `.sum()` on that counts
nulls per column. `.duplicated()` flags fully-duplicate rows, `.sum()` counts
them.

**When to use them here:** the basic structural checks from Session 2 —
run once, early, before trusting anything else about the data.

---

## `sklearn.metrics.confusion_matrix(..., normalize=...)`

**What it does:** like `value_counts(normalize=True)`, but for classifier
predictions. `normalize='true'` normalizes by row (true class), `'pred'` by
column (predicted class), `'all'` by the whole matrix.

**When to use it here:** Phase 3 baseline model evaluation — turns raw
prediction counts into rates, which is usually what you actually want to
read (e.g. "of all actual positives, what fraction did we catch" rather than
a raw count that means nothing without context).

---

## The `normalize` naming trap

The word `normalize` does **not** mean "convert to proportions" everywhere —
worth checking a function's docstring the first time, rather than assuming:

- `pandas.Timestamp.normalize()` / `DatetimeIndex.normalize()` — strips the
  time component down to midnight. A date-cleaning operation, unrelated to
  proportions.
- `sklearn.preprocessing.normalize()` — rescales feature vectors to unit norm
  (L1/L2). A feature-scaling operation, closer to standardization than to
  "proportion of total."

Three unrelated meanings behind one keyword, across pandas (counts),
pandas (dates), and sklearn (scaling).

---

## Non-destructive by default: `drop_duplicates()`, `dropna()`, `sort_values()`, etc.

**What it does:** most DataFrame methods **return a new DataFrame** and leave
the original object unchanged, unless you either capture the return value or
pass `inplace=True` (generally discouraged in modern pandas — capturing the
return value is the safer habit, since `inplace` can silently fail on a view
vs. a copy in some cases).

**The trap:** calling `df.drop_duplicates()` on its own line just displays the
deduplicated result and throws it away — `df` itself is still the original,
full-of-duplicates version. Any code on the *next* line that references `df`
is still operating on the untouched original, which can silently invalidate
whatever you were trying to check.

**What actually happened here:** ran `df.drop_duplicates()` alone, then
`df['treatment'].mean()` on the next line — got the same 0.85 as before
deduplication, which looked like "dedup didn't change the treatment ratio."
It actually meant "dedup never happened to `df` at all," since the result was
never captured.

**How to do it correctly** — capture the result, either into a new variable
or chained directly:
```python
df_dedup = df.drop_duplicates()
df_dedup['treatment'].mean()

# or, chained without a new variable:
df.drop_duplicates()['treatment'].mean()
```

**When to use it:** any time a check depends on "after removing/sorting/
transforming rows, does X change" — always re-verify you're reading the
transformed object, not the original.
