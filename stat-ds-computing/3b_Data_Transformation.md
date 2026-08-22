# 2.2 Joining, Reshaping & Missing Data

## Why this week matters

Week 3 focused on selecting observations and variables from a single DataFrame. Real data science projects often require something more complicated:

- information is split across several tables;
- datasets need to be stacked together;
- the same variable may appear across many columns;
- observations may be represented in wide or long form;
- data may be incomplete.

The central question for Week 4 is:

> **How do we transform multiple imperfect tables into one analysis-ready dataset without changing the meaning of the data?**

This week covers:

- `merge()` and database-style joins;
- `concat()` for stacking or combining DataFrames;
- wide vs. long data;
- `melt()` and `pivot()`;
- missing-data detection and basic strategies;
- validation of transformations.

---

# 1. Data rarely arrive as one perfect table

Suppose one table contains participant information:

```python
participants = pd.DataFrame({
    "id": [101, 102, 103],
    "age": [24, 31, 27],
    "group": ["A", "B", "A"]
})
```

and another contains outcomes:

```python
scores = pd.DataFrame({
    "id": [101, 102, 103],
    "score": [81, 94, 88]
})
```

To analyze score by age or group, we need to combine them.

Conceptually:

```text
participants            scores
-----------             ------
id age group            id score
101 24 A                101 81
102 31 B                102 94
103 27 A                103 88

          ↓ merge on id

id age group score
101 24 A     81
102 31 B     94
103 27 A     88
```

---

# 2. Keys

A **key** is a variable used to identify matching observations across tables.

Examples:

```text
participant_id
student_id
county_fips
date
hospital_id
```

A good key should have a clear interpretation.

Before joining, ask:

1. What variable identifies observations?
2. Is the key unique in either table?
3. Are there duplicate keys?
4. Are there missing keys?
5. Do key data types match?

A join can run successfully and still produce the wrong dataset.

---

# 3. `merge()`

pandas uses `merge()` for database-style joins.

```python
combined = participants.merge(
    scores,
    on="id"
)
```

Equivalent:

```python
combined = pd.merge(
    participants,
    scores,
    on="id"
)
```

The important question is:

> **Which rows should survive when the keys do not match?**

That is controlled by `how=`.

---

# 4. Inner join

```python
left.merge(
    right,
    on="id",
    how="inner"
)
```

An inner join keeps keys present in **both** tables.

Conceptually:

```text
Left IDs:   {1, 2, 3}
Right IDs:  {2, 3, 4}

Inner:      {2, 3}
```

Use an inner join when only matched observations should remain.

But note:

> An inner join can silently remove observations.

Always inspect row counts and unmatched keys.

---

# 5. Left join

```python
left.merge(
    right,
    on="id",
    how="left"
)
```

A left join preserves all observations in the left table.

```text
Left IDs:   {1, 2, 3}
Right IDs:  {2, 3, 4}

Left join:  {1, 2, 3}
```

Rows with no matching value on the right receive missing values.

This is often useful when the left table defines the study population.

---

# 6. Right and outer joins

A right join preserves the right table:

```python
left.merge(
    right,
    on="id",
    how="right"
)
```

An outer join keeps all keys appearing in either table:

```python
left.merge(
    right,
    on="id",
    how="outer"
)
```

Conceptually:

```text
Left:   {1, 2, 3}
Right:  {2, 3, 4}

Outer:  {1, 2, 3, 4}
```

Outer joins are useful for diagnosing mismatches.

---

# 7. Join logic as set logic

Join types correspond closely to set relationships:

```text
inner → intersection
left  → all left keys
right → all right keys
outer → union
```

This is a useful mental model because joining is fundamentally about deciding which entities belong in the resulting dataset.

---

# 8. Diagnosing joins with `indicator=True`

pandas can record where each merged row came from:

```python
diagnostic = left.merge(
    right,
    on="id",
    how="outer",
    indicator=True
)
```

This creates:

```text
_merge
```

with values such as:

```text
left_only
right_only
both
```

Then:

```python
diagnostic["_merge"].value_counts()
```

provides a quick audit of the join.

This is one of the most useful habits for reliable data engineering.

---

# 9. Validate join relationships

Suppose each participant should appear once in both tables.

We expect:

```text
one-to-one
```

pandas can validate this:

```python
left.merge(
    right,
    on="id",
    validate="one_to_one"
)
```

Other possibilities include:

```text
one_to_many
many_to_one
many_to_many
```

If the relationship violates your expectation, pandas can raise an error.

This converts an assumption into an executable check.

---

# 10. Duplicate keys can multiply rows

Consider:

```text
left
id visit
1  A
1  B

right
id treatment
1  control
1  active
```

A many-to-many merge creates four rows.

This is not necessarily wrong—but it may be disastrous if it was not intended.

Before merging:

```python
left["id"].duplicated().sum()
right["id"].duplicated().sum()
```

or:

```python
left["id"].value_counts()
```

Understand the cardinality before joining.

---

# 11. Joining on differently named keys

Sometimes keys have different names:

```python
patients.merge(
    outcomes,
    left_on="patient_id",
    right_on="id"
)
```

Afterward, consider whether both key columns are still needed.

Clear names reduce ambiguity.

---

# 12. `merge()` vs. `concat()`

Use:

```python
merge()
```

when tables should be matched based on keys.

Use:

```python
concat()
```

when objects should be stacked along an axis.

A useful distinction:

```text
merge  → match records
concat → stack objects
```

---

# 13. Concatenating rows

Suppose the same variables are collected in two semesters:

```python
fall = pd.DataFrame(...)
spring = pd.DataFrame(...)
```

Combine them:

```python
all_students = pd.concat(
    [fall, spring],
    axis=0,
    ignore_index=True
)
```

This stacks rows.

Conceptually:

```text
Fall
─────
rows

  +

Spring
──────
rows

  ↓

All rows
```

---

# 14. Concatenating columns

We can concatenate horizontally:

```python
pd.concat(
    [demographics, scores],
    axis=1
)
```

But this aligns rows using the index.

If the indexes do not represent the same entities, the result may be wrong.

When entities need to be matched by a key, `merge()` is usually clearer.

---

# 15. Wide and long data

Consider repeated measurements.

## Wide format

```text
id  score_pre  score_post
1      72          81
2      85          91
3      79          84
```

Each subject occupies one row.

## Long format

```text
id  time   score
1   pre      72
1   post     81
2   pre      85
2   post     91
3   pre      79
3   post     84
```

Each subject can occupy multiple rows.

Neither format is universally better.

The appropriate form depends on the operation or statistical model.

---

# 16. Wide → long with `melt()`

```python
long = wide.melt(
    id_vars="id",
    value_vars=["score_pre", "score_post"],
    var_name="time",
    value_name="score"
)
```

`melt()` turns measured columns into rows while preserving identifier variables.

Conceptually:

```text
columns representing repeated measurements
                 ↓
           become rows
```

This is often useful for repeated-measures analysis and visualization.

---

# 17. Cleaning after `melt()`

After melting:

```text
score_pre
score_post
```

may appear in the `time` column.

We can transform them:

```python
long["time"] = long["time"].str.replace(
    "score_",
    "",
    regex=False
)
```

Result:

```text
pre
post
```

Reshaping and cleaning often occur together.

---

# 18. Long → wide with `pivot()`

Suppose:

```text
id time score
1  pre   72
1  post  81
2  pre   85
2  post  91
```

Use:

```python
wide = long.pivot(
    index="id",
    columns="time",
    values="score"
)
```

This moves values of `time` into columns.

---

# 19. `pivot()` requires unique combinations

For `pivot()` to work cleanly, each combination of:

```text
index × column
```

must correspond to one value.

For example:

```text
id=1, time=pre
```

should identify one score.

If duplicate combinations exist, `pivot()` raises an error.

That error is useful: it indicates the data structure does not satisfy the assumption.

---

# 20. `pivot_table()`

When repeated combinations must be aggregated, use:

```python
pd.pivot_table(
    data,
    index="id",
    columns="time",
    values="score",
    aggfunc="mean"
)
```

This is fundamentally different from `pivot()`.

```text
pivot()
    → reshape unique records

pivot_table()
    → reshape + aggregate
```

Do not aggregate duplicates merely to make an error disappear. First determine why duplicates exist.

---

# 21. Missing data

pandas typically represents missing numeric values using values such as `NaN`, with newer nullable data types also supporting `pd.NA`.

Detect missingness:

```python
df.isna()
```

Count by column:

```python
df.isna().sum()
```

Percentage missing:

```python
df.isna().mean() * 100
```

Missing values require statistical decisions, not merely coding decisions.

---

# 22. Why values are missing matters

A value may be missing because:

- it was not measured;
- a participant skipped a question;
- equipment failed;
- the variable did not apply;
- a join failed to find a matching record;
- the value was incorrectly encoded;
- the observation was intentionally censored.

Those mechanisms have different interpretations.

Before choosing a strategy, ask:

> **Why is this value missing?**

---

# 23. Dropping missing values

Remove rows containing any missing values:

```python
df.dropna()
```

Require specific variables:

```python
df.dropna(
    subset=["age", "score"]
)
```

This can be appropriate in some settings, but it changes the analysis population.

Always record how many rows were removed:

```python
before = len(df)

complete = df.dropna(
    subset=["score"]
)

after = len(complete)

before - after
```

---

# 24. Filling missing values

Simple replacement:

```python
df["score"] = df["score"].fillna(0)
```

Mean replacement:

```python
df["score"] = df["score"].fillna(
    df["score"].mean()
)
```

But technical possibility does not imply statistical validity.

For example, replacing a missing score with zero asserts that missing means zero.

Replacing with the mean changes the variance and distribution.

Use imputation strategies only when their assumptions are defensible.

---

# 25. Missingness introduced by joins

Suppose:

```python
participants.merge(
    outcomes,
    on="id",
    how="left"
)
```

Some outcome values become missing.

That may not mean the outcome was originally missing.

It may mean:

> No matching outcome record existed.

This is why join auditing and missing-data analysis are connected.

---

# 26. Missing-data indicators

Sometimes it is useful to create an explicit indicator:

```python
df["score_missing"] = df["score"].isna()
```

This can help with:

- diagnostics;
- reporting;
- examining patterns of missingness.

For example:

```python
pd.crosstab(
    df["group"],
    df["score_missing"]
)
```

---

# 27. Transformation invariants

A good data transformation should preserve facts we expect to remain true.

Examples:

```text
Number of unique participants
Expected key uniqueness
Expected groups
Expected date range
Expected number of repeated measures
```

We can write checks:

```python
assert df["id"].nunique() == 100
```

or:

```python
assert merged["id"].is_unique
```

Data engineering becomes safer when assumptions are executable.

---

# 28. Validate before and after

Before a merge:

```python
left.shape
right.shape

left["id"].nunique()
right["id"].nunique()
```

After:

```python
merged.shape
merged["id"].nunique()
merged.isna().sum()
```

For an outer diagnostic merge:

```python
diagnostic["_merge"].value_counts()
```

The principle is:

> **Never trust a transformation simply because it ran.**

---

# 29. A professional wrangling workflow

A useful workflow is:

```text
Inspect
   ↓
Define keys
   ↓
Check uniqueness
   ↓
Merge / concatenate
   ↓
Audit row counts
   ↓
Reshape
   ↓
Inspect missingness
   ↓
Apply justified strategy
   ↓
Validate
   ↓
Document
```

---

# 30. Key ideas

By the end of Week 4, you should be able to explain:

1. What a join key represents.
2. Inner, left, right, and outer joins.
3. How joins relate to set logic.
4. Why duplicate keys matter.
5. How `indicator=True` helps diagnose joins.
6. Why `validate=` is useful.
7. The distinction between `merge()` and `concat()`.
8. Wide vs. long data.
9. How `melt()` converts wide data to long.
10. How `pivot()` converts long data to wide.
11. Why `pivot()` and `pivot_table()` are not identical.
12. How missing data can arise naturally or through transformations.
13. Why missing-data strategies require statistical justification.
14. Why transformations should be validated.

---

# 31. Recommended reading

## pandas — How to combine data from multiple tables

The official pandas tutorial gives a clear introduction to both concatenation and database-style merges.

https://pandas.pydata.org/docs/getting_started/intro_tutorials/08_combine_dataframes.html

## pandas — Merge, join, concatenate and compare

The detailed user guide for combining pandas objects.

https://pandas.pydata.org/docs/user_guide/merging.html

## pandas — Reshaping and pivot tables

Official documentation for `pivot()`, `pivot_table()`, `melt()`, `stack()`, and related reshaping tools.

https://pandas.pydata.org/docs/user_guide/reshaping.html

## pandas — Working with missing data

Official documentation for detecting, representing, removing, and filling missing values.

https://pandas.pydata.org/docs/user_guide/missing_data.html

## Python for Data Analysis — Wes McKinney

The chapters on data wrangling, combining datasets, and reshaping provide an excellent extended reference.

https://wesmckinney.com/book/

---

# 32. YouTube recommendations

## 1. Chart Explorers — "How to combine DataFrames in Pandas | Merge, Join, Concat, & Append"

A practical overview of combining DataFrames using merges, joins, and concatenation. It also demonstrates inner, left, right, and outer join logic.

**Recommended use:** Watch before or after the merge portion of the hands-on tutorial.

[Watch on YouTube](https://www.youtube.com/watch?v=wzN1UyfRSWI)

---

## 2. Rob Mulla — "Melt & Pivot Data with Pandas"

A concise demonstration of reshaping pandas DataFrames with `melt()` and `pivot()`. This maps directly onto the wide-to-long and long-to-wide material in this week.

**Recommended use:** Watch before the reshaping exercises.

[Watch on YouTube](https://www.youtube.com/watch?v=GXluB4yDlCY)

---

## 3. Corey Schafer — "Python Pandas Tutorial (Part 9): Cleaning Data — Casting Datatypes and Handling Missing Values"

A detailed practical tutorial on cleaning pandas data, including detecting, dropping, and filling missing values.

**Recommended use:** Watch alongside the missing-data section of the tutorial.

[Watch on YouTube](https://www.youtube.com/watch?v=KdmPHEnPJPs)

---

# 33. Week 4 takeaway

The central lesson is:

> **Data wrangling is the process of preserving meaning while changing structure.**

The progression is:

```text
Multiple tables
      ↓
Keys
      ↓
Merge / concatenate
      ↓
Audit
      ↓
Wide ↔ long
      ↓
Missingness
      ↓
Statistical decisions
      ↓
Validation
      ↓
Analysis-ready data
```

Next week we will move into **input/output and external data sources**, including SQL, APIs, JSON, HTML, and binary data formats.
