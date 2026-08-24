# 2.2 Activity

## Joining, Reshaping & Missing Data

**Tools:** Python, pandas, VS Code and/or Jupyter

## Course computing environment: GitHub Codespaces

For STA 556, the **officially supported computing environment is the course GitHub Codespace**. This gives everyone the same Linux/Python environment whether you are using a Mac, a Windows PC, or a university computer.

Before beginning this activity:

1. Open the repository for your course/assignment on GitHub.
2. Open or create its Codespace.
3. In VS Code, make sure the **repository root** is the folder open in the Explorer.
4. Open an integrated terminal and run:

```bash
pwd
git status
```

Your working directory should be the repository under `/workspaces/...`, and `git status` should recognize the repository.

Unless an activity explicitly says otherwise:

- run terminal commands from the **repository root**;
- use repository-relative paths such as `data/...`, `src/...`, and `notebooks/...`;
- use `python` for Python commands;
- do **not** run `git init` inside the course repository;
- do **not** install packages manually just because an import fails—first check the course environment and `requirements.txt`;
- commit and push meaningful work regularly.

If this is your first time using Codespaces, read `0_Codespaces_Introduction.md` before continuing.

---

## Learning objectives

By the end of this tutorial, you should be able to:

- combine tables using `merge()`;
- distinguish inner, left, right, and outer joins;
- identify and diagnose unmatched records;
- detect duplicate keys and validate join cardinality;
- combine datasets using `concat()`;
- convert data between wide and long form;
- use `melt()`, `pivot()`, and `pivot_table()`;
- detect and summarize missing data;
- apply simple missing-data strategies critically;
- validate a multi-step wrangling pipeline.

---

# Part 0 — Set up

Create:

```text
notebooks/week04_wrangle_part2.ipynb
```

At the top:

```python
import pandas as pd
import numpy as np
```

---

# Part 1 — Create two related tables

Create participant information:

```python
participants = pd.DataFrame({
    "id": [101, 102, 103, 104, 105, 106],
    "age": [24, 31, 27, 19, 42, 35],
    "group": ["A", "B", "A", "B", "A", "B"]
})
```

Create scores:

```python
scores = pd.DataFrame({
    "id": [101, 102, 103, 105, 106, 107],
    "score": [81, 94, 88, 91, 85, 79]
})
```

Inspect:

```python
participants
scores
```

### Questions

1. Which IDs occur in both tables?
2. Which ID occurs only in `participants`?
3. Which ID occurs only in `scores`?
4. What do you expect an inner join to contain?

Do not run a merge yet.

---

# Part 2 — Inner join

Run:

```python
inner = participants.merge(
    scores,
    on="id",
    how="inner"
)
```

Inspect:

```python
inner
inner.shape
```

### Questions

1. Which IDs were removed?
2. Why?
3. Did the number of rows match your prediction?

---

# Part 3 — Left join

Run:

```python
left = participants.merge(
    scores,
    on="id",
    how="left"
)
```

Inspect:

```python
left
```

### Questions

1. Which table defines the resulting population?
2. What happened to ID 104?
3. Why is its score missing?

---

# Part 4 — Outer join

Run:

```python
outer = participants.merge(
    scores,
    on="id",
    how="outer"
)
```

Inspect:

```python
outer
```

### Challenge

Before running a right join, predict which IDs it will contain.

Then:

```python
right = participants.merge(
    scores,
    on="id",
    how="right"
)
```

---

# Part 5 — Diagnose joins with `indicator=True`

Run:

```python
diagnostic = participants.merge(
    scores,
    on="id",
    how="outer",
    indicator=True
)
```

Inspect:

```python
diagnostic
```

Then:

```python
diagnostic["_merge"].value_counts()
```

### Questions

1. How many rows matched?
2. How many occurred only in the participant table?
3. How many occurred only in the score table?

### Challenge

Extract only unmatched records.

---

# Part 6 — Check key uniqueness

Run:

```python
participants["id"].is_unique
scores["id"].is_unique
```

Then:

```python
participants["id"].duplicated().sum()
scores["id"].duplicated().sum()
```

### Question

Why should this be checked before a one-to-one join?

---

# Part 7 — Validate the merge

Run:

```python
validated = participants.merge(
    scores,
    on="id",
    how="left",
    validate="one_to_one"
)
```

Now create a bad score table:

```python
duplicate_scores = pd.DataFrame({
    "id": [101, 101, 102],
    "score": [81, 83, 94]
})
```

Try:

```python
participants.merge(
    duplicate_scores,
    on="id",
    validate="one_to_one"
)
```

### Question

Why is the error valuable?

---

# Part 8 — Many-to-many multiplication

Create:

```python
visits = pd.DataFrame({
    "id": [1, 1, 2],
    "visit": ["pre", "post", "pre"]
})

treatments = pd.DataFrame({
    "id": [1, 1, 2],
    "treatment": ["A", "B", "A"]
})
```

Run:

```python
visits.merge(
    treatments,
    on="id"
)
```

### Questions

1. Why does ID 1 generate four rows?
2. Was information duplicated?
3. Under what circumstances could this be correct?

---

# Part 9 — Concatenate rows

Create two semester datasets:

```python
fall = pd.DataFrame({
    "id": [1, 2, 3],
    "semester": ["Fall"] * 3,
    "score": [80, 85, 90]
})

spring = pd.DataFrame({
    "id": [4, 5, 6],
    "semester": ["Spring"] * 3,
    "score": [78, 92, 88]
})
```

Combine:

```python
all_scores = pd.concat(
    [fall, spring],
    axis=0,
    ignore_index=True
)
```

Inspect:

```python
all_scores
```

### Question

Why is `concat()` more natural than `merge()` here?

---

# Part 10 — Concatenating mismatched columns

Create:

```python
fall = pd.DataFrame({
    "id": [1, 2],
    "score": [80, 85]
})

spring = pd.DataFrame({
    "id": [3, 4],
    "score": [90, 92],
    "campus": ["Flagstaff", "Flagstaff"]
})
```

Run:

```python
pd.concat(
    [fall, spring],
    ignore_index=True
)
```

### Question

What does pandas do with the column that does not exist in `fall`?

---

# Part 11 — Wide data

Create repeated measurements:

```python
wide = pd.DataFrame({
    "id": [101, 102, 103, 104],
    "group": ["A", "B", "A", "B"],
    "score_pre": [72, 85, 79, 88],
    "score_post": [81, 91, 84, 90]
})
```

Inspect:

```python
wide
```

### Question

How many rows represent each participant?

How are repeated measurements represented?

---

# Part 12 — Wide to long with `melt()`

Run:

```python
long = wide.melt(
    id_vars=["id", "group"],
    value_vars=["score_pre", "score_post"],
    var_name="time",
    value_name="score"
)
```

Inspect:

```python
long
```

### Questions

1. How many rows now represent each participant?
2. What happened to the original score columns?
3. What does `id_vars` mean?
4. What does `value_vars` mean?

---

# Part 13 — Clean the melted variable

Run:

```python
long["time"] = (
    long["time"]
    .str.replace("score_", "", regex=False)
)
```

Inspect:

```python
long
```

### Challenge

Change:

```text
pre
post
```

to:

```text
baseline
followup
```

using:

```python
.replace()
```

---

# Part 14 — Long to wide with `pivot()`

Start from `long`.

Run:

```python
wide_again = long.pivot(
    index=["id", "group"],
    columns="time",
    values="score"
)
```

Inspect:

```python
wide_again
```

Reset the index:

```python
wide_again = wide_again.reset_index()
```

### Question

Did you recover the same information as the original wide dataset?

---

# Part 15 — Duplicate combinations and `pivot()`

Create:

```python
repeated = pd.DataFrame({
    "id": [1, 1, 1, 2, 2],
    "time": ["pre", "pre", "post", "pre", "post"],
    "score": [70, 74, 82, 80, 88]
})
```

Try:

```python
repeated.pivot(
    index="id",
    columns="time",
    values="score"
)
```

### Question

Why does this fail?

Which assumption is violated?

---

# Part 16 — `pivot_table()`

Now use:

```python
summary = repeated.pivot_table(
    index="id",
    columns="time",
    values="score",
    aggfunc="mean"
)
```

Inspect:

```python
summary
```

### Reflection

Why should you investigate duplicate records before deciding that taking the mean is appropriate?

---

# Part 17 — Create missing data

Create:

```python
clinical = pd.DataFrame({
    "id": [1, 2, 3, 4, 5, 6],
    "age": [24, 31, np.nan, 45, 28, 39],
    "group": ["A", "B", "A", "B", None, "A"],
    "score": [81, np.nan, 77, 92, 88, np.nan]
})
```

Inspect:

```python
clinical
```

Then:

```python
clinical.isna()
```

and:

```python
clinical.isna().sum()
```

Calculate percentages:

```python
clinical.isna().mean() * 100
```

---

# Part 18 — Identify rows with missing data

Find rows with missing score:

```python
clinical.loc[
    clinical["score"].isna()
]
```

Find complete scores:

```python
clinical.loc[
    clinical["score"].notna()
]
```

Find rows where either age OR score is missing:

```python
clinical.loc[
    clinical[["age", "score"]]
    .isna()
    .any(axis=1)
]
```

### Challenge

Find rows where both age and score are observed.

---

# Part 19 — Drop missing observations

Run:

```python
complete_score = clinical.dropna(
    subset=["score"]
)
```

Compare:

```python
len(clinical)
len(complete_score)
```

Now require both age and score:

```python
complete_cases = clinical.dropna(
    subset=["age", "score"]
)
```

### Questions

1. How many observations were removed?
2. Has the analysis population changed?
3. What assumptions might make complete-case analysis problematic?

---

# Part 20 — Filling missing values

Create a copy:

```python
filled = clinical.copy()
```

Fill scores with the mean:

```python
filled["score"] = filled["score"].fillna(
    filled["score"].mean()
)
```

Compare:

```python
clinical["score"].describe()
filled["score"].describe()
```

### Questions

1. Did the mean change?
2. Did the standard deviation change?
3. Why?
4. Is mean imputation automatically a good statistical strategy?

---

# Part 21 — Missingness introduced by a join

Return to:

```python
participants
scores
```

Create:

```python
joined = participants.merge(
    scores,
    on="id",
    how="left"
)
```

Find:

```python
joined.loc[
    joined["score"].isna()
]
```

### Question

Does a missing score here necessarily mean that the participant failed to provide a score?

What other explanation exists?

---

# Part 22 — Missingness indicator

Create:

```python
clinical["score_missing"] = (
    clinical["score"].isna()
)
```

Inspect:

```python
clinical
```

Now:

```python
pd.crosstab(
    clinical["group"],
    clinical["score_missing"]
)
```

### Reflection

What question does this table help you investigate?

---

# Part 23 — Build a complete wrangling workflow

Create demographic data:

```python
demographics = pd.DataFrame({
    "id": [101, 102, 103, 104, 105],
    "age": [24, 31, 27, 42, 36],
    "group": ["A", "B", "A", "B", "A"]
})
```

Create repeated outcomes:

```python
outcomes = pd.DataFrame({
    "id": [101, 102, 103, 105],
    "pre": [72, 80, 85, 78],
    "post": [81, 87, np.nan, 86]
})
```

Your task is to produce a long-format analysis dataset containing:

```text
id
age
group
time
score
```

with all demographic participants preserved.

---

# Part 24 — Step 1: audit the keys

Check:

```python
demographics["id"].is_unique
outcomes["id"].is_unique
```

Compare sets of IDs:

```python
set(demographics["id"]) - set(outcomes["id"])
```

and:

```python
set(outcomes["id"]) - set(demographics["id"])
```

Write down what you expect the merge to do.

---

# Part 25 — Step 2: merge

Use:

```python
combined = demographics.merge(
    outcomes,
    on="id",
    how="left",
    validate="one_to_one",
    indicator=True
)
```

Inspect:

```python
combined
combined["_merge"].value_counts()
```

Remove the diagnostic column after checking:

```python
combined = combined.drop(
    columns="_merge"
)
```

---

# Part 26 — Step 3: reshape

Convert to long format:

```python
analysis = combined.melt(
    id_vars=["id", "age", "group"],
    value_vars=["pre", "post"],
    var_name="time",
    value_name="score"
)
```

Inspect:

```python
analysis
```

---

# Part 27 — Step 4: audit missingness

Run:

```python
analysis.isna().sum()
```

Then:

```python
analysis.loc[
    analysis["score"].isna()
]
```

### Questions

For each missing score, determine whether it arose because:

1. an outcome record was absent entirely; or
2. an outcome row existed but a particular measurement was missing.

These are different data problems.

---

# Part 28 — Step 5: define an analysis dataset

Suppose the analysis requires observed scores.

Create:

```python
observed = analysis.dropna(
    subset=["score"]
)
```

Record:

```python
n_before = len(analysis)
n_after = len(observed)
n_removed = n_before - n_after
```

Print these quantities.

### Reflection

Why should this information appear in an analysis report?

---

# Part 29 — Validate the transformation

Check:

```python
analysis["id"].nunique()
```

Compare with:

```python
demographics["id"].nunique()
```

Check time values:

```python
analysis["time"].value_counts()
```

Check participant-time uniqueness:

```python
analysis.duplicated(
    subset=["id", "time"]
).sum()
```

Add assertions:

```python
assert (
    analysis["id"].nunique()
    == demographics["id"].nunique()
)

assert (
    analysis.duplicated(
        subset=["id", "time"]
    ).sum()
    == 0
)
```

---

# Part 30 — Mini challenge: three files

Imagine three monthly extracts:

```python
jan = pd.DataFrame({
    "id": [1, 2],
    "month": ["Jan", "Jan"],
    "score": [80, 85]
})

feb = pd.DataFrame({
    "id": [1, 2],
    "month": ["Feb", "Feb"],
    "score": [82, 88]
})

mar = pd.DataFrame({
    "id": [1, 2],
    "month": ["Mar", "Mar"],
    "score": [84, np.nan]
})
```

### Tasks

1. Concatenate the three tables.
2. Verify the number of rows.
3. Identify missing scores.
4. Pivot into wide form with one row per participant.
5. Convert back to long form.
6. Verify that the structure is equivalent to the original combined data.

---

# Part 31 — Written reasoning challenge

For each scenario, choose the most appropriate operation.

### A

Two tables contain the same variables for different months.

```text
merge or concat?
```

### B

One table contains demographics and another contains outcomes linked by participant ID.

```text
merge or concat?
```

### C

Repeated measurements are stored in `score_pre`, `score_post`, and `score_followup`.

```text
melt or pivot?
```

### D

Long data should become one row per participant.

```text
melt or pivot?
```

### E

You need every participant from the enrollment table, whether or not an outcome exists.

```text
inner or left join?
```

Explain each answer.

---

# Part 32 — Git checkpoint

Run:

```bash
git status
```

Stage:

```bash
git add .
```

Commit:

```bash
git commit -m "Complete Week 4 joining reshaping and missing data"
```

Push:

```bash
git push
```

---

# Part 33 — Final reflection

Answer in Markdown.

### 1. Merge

What is the conceptual difference between an inner join and a left join?

### 2. Keys

Why should you inspect duplicate keys before merging?

### 3. Validation

What does `validate="one_to_one"` protect against?

### 4. Concatenation

When is `concat()` more appropriate than `merge()`?

### 5. Reshaping

What is the difference between wide and long data?

### 6. Pivoting

Why might `pivot()` fail when `pivot_table()` succeeds?

### 7. Missingness

Why should missing values introduced by a join be interpreted differently from missing values already present in a source table?

### 8. Statistical practice

Why is dropping missing observations a statistical decision rather than only a programming operation?

---

# Completion checklist

- [ ] Created Week 4 notebook
- [ ] Performed an inner join
- [ ] Performed a left join
- [ ] Performed a right join
- [ ] Performed an outer join
- [ ] Used `indicator=True`
- [ ] Checked duplicate keys
- [ ] Used `validate=`
- [ ] Explored a many-to-many merge
- [ ] Concatenated rows
- [ ] Examined mismatched columns during concatenation
- [ ] Created wide-format data
- [ ] Used `melt()`
- [ ] Used `pivot()`
- [ ] Used `pivot_table()`
- [ ] Detected missing values
- [ ] Calculated missing-data percentages
- [ ] Used `dropna()`
- [ ] Used `fillna()`
- [ ] Compared distributions before and after filling
- [ ] Created a missingness indicator
- [ ] Completed the full merge → reshape → missingness workflow
- [ ] Added validation assertions
- [ ] Completed the monthly-data challenge
- [ ] Committed work to Git
- [ ] Pushed work to GitHub

---

# What you should now understand

```text
Separate datasets
       ↓
Identify keys
       ↓
Check relationships
       ↓
Merge / concatenate
       ↓
Audit
       ↓
Reshape
       ↓
Inspect missingness
       ↓
Make statistical decisions
       ↓
Validate
       ↓
Analysis-ready data
```

Next week we will work with **external data sources**, including SQL databases, APIs, JSON, HTML, and binary file formats.
