# 2.1 DataFrames, Indexing, Slicing & Filtering

## Why this week matters

In Week 2, we worked with Python's built-in data structures. This week we move to the central structure used for tabular statistical computing in Python: the **pandas DataFrame**.

The main question is:

> **How do we tell the computer exactly which observations and variables belong in an analysis?**

The approved course schedule identifies Week 3 as the first data-engineering module, covering **pandas DataFrames, indexing, slicing, and filtering**.

---

# 1. From Python objects to tabular data

Last week we could represent observations as a list of dictionaries:

```python
participants = [
    {"id": 101, "age": 24, "group": "A", "score": 81},
    {"id": 102, "age": 31, "group": "B", "score": 94},
    {"id": 103, "age": 27, "group": "A", "score": 88}
]
```

Using pandas:

```python
import pandas as pd

df = pd.DataFrame(participants)
```

we obtain a labeled table.

```text
Python objects
      ↓
lists / dictionaries
      ↓
pandas Series
      ↓
pandas DataFrame
      ↓
statistical data manipulation
```

---

# 2. What is a DataFrame?

A pandas `DataFrame` is a two-dimensional tabular data structure with labeled rows and columns.

Conceptually:

```text
             age   group   score
subject_id
101           24      A      81
102           31      B      94
103           27      A      88
104           19      B      72
```

A DataFrame contains:

- observations (rows);
- variables (columns);
- an index (row labels);
- column labels;
- values;
- data types.

This structure closely matches the rectangular datasets used throughout statistics and data science.

---

# 3. Creating a DataFrame

```python
import pandas as pd

data = {
    "id": [101, 102, 103, 104],
    "age": [24, 31, 27, 19],
    "group": ["A", "B", "A", "B"],
    "score": [81, 94, 88, 72]
}

df = pd.DataFrame(data)
```

Useful attributes:

```python
df.shape
df.columns
df.index
df.dtypes
```

Useful methods:

```python
df.head()
df.tail()
df.info()
df.describe()
```

---

# 4. Inspect before you analyze

A professional workflow starts by asking:

> **What exactly is in this dataset?**

Before analysis, determine:

1. How many observations are present?
2. How many variables?
3. Which variables are numeric?
4. Which are categorical or text?
5. Are names and data types what we expected?
6. Are values missing?

Do not begin modeling before understanding the structure of the data.

---

# 5. Series vs. DataFrame

A DataFrame contains columns represented as pandas `Series` objects.

```python
df["age"]
```

returns a Series.

```python
df[["age"]]
```

returns a DataFrame.

Compare:

```python
type(df["age"])
type(df[["age"]])
```

This distinction becomes important because Series and DataFrames behave differently in some operations.

---

# 6. Selecting columns

Select one column:

```python
df["age"]
```

Select multiple columns:

```python
df[["age", "score"]]
```

Think of this as choosing the **variables** required for an analysis.

---

# 7. The DataFrame index

Every DataFrame has an index.

Initially:

```python
df.index
```

may return a default integer index.

We can create a meaningful index:

```python
df = df.set_index("id")
```

Now the row labels are participant IDs.

---

# 8. `.loc` and `.iloc`

Two essential pandas indexers are:

```python
.loc
```

and:

```python
.iloc
```

Remember:

```text
loc  → labels
iloc → integer positions
```

---

# 9. Selecting by position with `.iloc`

```python
df.iloc[0]
```

means:

> Give me the first row.

```python
df.iloc[0:3]
```

selects the first three rows.

Rows and columns can be selected together:

```python
df.iloc[0:3, 0:2]
```

General form:

```python
df.iloc[row_positions, column_positions]
```

---

# 10. Selecting by label with `.loc`

If `id` is the index:

```python
df.loc[101]
```

means:

> Give me the row labeled 101.

Several labels:

```python
df.loc[[101, 103]]
```

Rows and columns:

```python
df.loc[
    [101, 103],
    ["age", "score"]
]
```

General form:

```python
df.loc[row_labels, column_labels]
```

---

# 11. `.loc` vs. `.iloc`

Suppose the row labels are:

```text
101
205
310
412
```

Then:

```python
df.loc[205]
```

selects the row **labeled** `205`.

```python
df.iloc[1]
```

selects the **second row by position**.

The commands may identify the same observation, but they express different instructions.

---

# 12. Slicing

Position-based slicing follows ordinary Python conventions:

```python
df.iloc[1:4]
```

includes positions 1, 2, and 3, but excludes 4.

Label-based slices behave differently:

```python
df.loc[101:103]
```

typically include both endpoint labels when they exist.

This difference is easy to overlook.

---

# 13. Selecting rows and columns together

A very useful general pattern is:

```python
df.loc[rows, columns]
```

For example:

```python
df.loc[:, ["age", "score"]]
```

means:

> All observations, but only age and score.

The comma separates **which observations** from **which variables**.

---

# 14. Boolean conditions

Consider:

```python
df["score"] > 80
```

This produces a Boolean Series such as:

```text
id
101     True
102     True
103     True
104    False
```

We can then use it to select observations:

```python
df.loc[df["score"] > 80]
```

---

# 15. Boolean filtering as statistical logic

Statistically, the expression:

```python
df["score"] > 80
```

acts like an indicator:

```text
I(score_i > 80)
```

For each observation:

```text
True  → include
False → exclude
```

Thus:

```text
D* = {i : score_i > 80}
```

Filtering is therefore an implementation of a statistical inclusion rule.

---

# 16. Filtering with one condition

Examples:

```python
df.loc[df["age"] >= 30]
```

```python
df.loc[df["group"] == "A"]
```

```python
df.loc[df["score"] >= 85]
```

General form:

```python
df.loc[condition]
```

---

# 17. Multiple conditions

Use `&` for AND:

```python
df.loc[
    (df["group"] == "A") &
    (df["score"] >= 85)
]
```

Use `|` for OR:

```python
df.loc[
    (df["group"] == "A") |
    (df["score"] >= 90)
]
```

Use `~` for NOT:

```python
df.loc[
    ~(df["group"] == "A")
]
```

Use parentheses around each condition.

---

# 18. Why not `and` and `or`?

This does not work as intended:

```python
(df["age"] >= 30) and (df["score"] >= 85)
```

Each comparison produces an entire Series of Boolean values.

For element-wise Boolean logic in pandas, use:

```python
&
|
~
```

---

# 19. Filter rows and choose variables together

Suppose we want:

> Participants scoring at least 85, showing only their group and score.

```python
df.loc[
    df["score"] >= 85,
    ["group", "score"]
]
```

Think:

```text
WHO?
    score >= 85

WHAT?
    group, score
```

---

# 20. Filtering categorical values with `.isin()`

To select multiple categories:

```python
df.loc[
    df["group"].isin(["A", "C"])
]
```

This is often clearer than a long sequence of OR conditions.

---

# 21. Missing values

Real datasets contain missing values.

Inspect:

```python
df.isna()
```

Count missing values:

```python
df.isna().sum()
```

Select rows with missing scores:

```python
df.loc[df["score"].isna()]
```

Select rows with observed scores:

```python
df.loc[df["score"].notna()]
```

An important statistical principle is:

> **Missingness is data.**

Missing-data decisions should be explicit and documented.

---

# 22. Assignment with `.loc`

`.loc` can also modify selected observations.

```python
df["performance"] = "medium"
```

Then:

```python
df.loc[
    df["score"] >= 90,
    "performance"
] = "high"
```

This clearly identifies both the rows to modify and the target column.

---

# 23. Avoid chained assignment

Avoid:

```python
df[df["score"] > 80]["score"] = 100
```

Prefer:

```python
df.loc[
    df["score"] > 80,
    "score"
] = 100
```

The `.loc` version is more explicit and avoids ambiguity around intermediate objects.

---

# 24. Filtering defines the analysis population

Consider:

```python
analysis_df = df.loc[
    (df["age"] >= 18) &
    (df["group"] == "A") &
    (df["score"].notna())
]
```

This code defines:

> Adults in group A with observed scores.

That is not merely data manipulation. It is part of the **statistical analysis specification**.

---

# 25. Separate criteria from selection

For more complicated analyses:

```python
eligible = (
    (df["age"] >= 30) &
    (df["score"] >= 85)
)
```

Then:

```python
analysis_df = df.loc[
    eligible,
    ["age", "group", "score"]
]
```

Naming the criterion can make the analysis easier to read, audit, and modify.

---

# 26. Always inspect the result

Code can execute successfully and still produce the wrong subset.

After filtering, inspect:

```python
analysis_df.shape
analysis_df.head()
analysis_df.index
analysis_df.describe()
```

Ask:

> Does this dataset contain exactly the observations I intended?

---

# 27. Common mistakes

### Mistake 1 — Confusing `.loc` and `.iloc`

```text
loc  → labels
iloc → positions
```

### Mistake 2 — Forgetting parentheses

Wrong:

```python
df["age"] >= 30 & df["score"] >= 85
```

Right:

```python
(df["age"] >= 30) & (df["score"] >= 85)
```

### Mistake 3 — Using `and` / `or`

Use `&`, `|`, and `~` with pandas Boolean Series.

### Mistake 4 — Confusing Series and DataFrames

```python
df["age"]      # Series
df[["age"]]    # DataFrame
```

### Mistake 5 — Chained assignment

Prefer:

```python
df.loc[condition, "column"] = value
```

---

# 28. Connection to the rest of STA 556

This week's work forms the foundation for later data-engineering tasks:

```text
DataFrame
   ↓
Indexing
   ↓
Filtering
   ↓
Joining
   ↓
Reshaping
   ↓
Missing-data strategies
   ↓
External data
   ↓
Statistical analysis
```

The underlying question remains:

> **Which observations and variables are we operating on?**

---

# 29. Key ideas

By the end of Week 3, you should be able to explain:

1. What a pandas DataFrame represents.
2. The relationship between a DataFrame and Series.
3. Why data should be inspected before analysis.
4. How to select one or several columns.
5. The role of the DataFrame index.
6. The difference between `.loc` and `.iloc`.
7. How positional and label slicing differ.
8. How Boolean conditions work.
9. How to combine multiple conditions.
10. How `.isin()` supports categorical filtering.
11. How `.isna()` and `.notna()` identify missingness.
12. How to assign values safely with `.loc`.
13. Why chained assignment should be avoided.
14. Why filtering is part of the statistical analysis specification.

---

# 30. Recommended reading

## pandas — 10 Minutes to pandas

A high-level introduction to DataFrame creation, viewing data, selection, and basic operations.

https://pandas.pydata.org/docs/user_guide/10min.html

## pandas — Indexing and selecting data

The main pandas reference for `.loc`, `.iloc`, slicing, Boolean indexing, and assignment.

https://pandas.pydata.org/docs/user_guide/indexing.html

## pandas — How do I select a subset of a DataFrame?

An accessible official tutorial focused on selecting columns, filtering rows, and using `.loc` and `.iloc`.

https://pandas.pydata.org/docs/getting_started/intro_tutorials/03_subset_data.html

## Python for Data Analysis — Wes McKinney

The chapters on pandas and data manipulation are especially relevant for Weeks 3 and 4.

https://wesmckinney.com/book/

---

# 31. YouTube recommendations

## 1. Data School — "How do I filter rows of a pandas DataFrame by column value?"

A clear step-by-step explanation of pandas Boolean filtering. The video builds the filtering syntax from ordinary Python logic, making it particularly useful for understanding what a Boolean mask represents.

**Recommended use:** Watch before or after the Boolean-filtering portion of the tutorial.

[Watch on YouTube](https://www.youtube.com/watch?v=2AFGPdNn4FM)

---

## 2. StrataScratch — "Pandas iloc, loc and Dataframes — Beginner Python Pandas Tutorial #1"

This tutorial introduces Series and DataFrames and demonstrates how `.loc` and `.iloc` select subsets of tabular data.

**Recommended use:** Watch alongside the sections on indexing, slicing, `.loc`, and `.iloc`.

[Watch on YouTube](https://www.youtube.com/watch?v=bM7LSQIKYs8)

---

## 3. Sundeep Saradhi Kanthety — "Data Filtering (Using Conditions) in Pandas"

A practical walkthrough of conditional filtering in pandas, useful for reinforcing Boolean conditions and filtering syntax after students have attempted the exercises themselves.

**Recommended use:** Optional reinforcement after the hands-on filtering exercises.

[Watch on YouTube](https://www.youtube.com/watch?v=Sz_iXvh25Ew)

---

# 32. Week 3 takeaway

The central lesson is:

> **Indexing and filtering are how we translate statistical inclusion rules into executable code.**

```text
DataFrame
   ↓
Inspect
   ↓
Select variables
   ↓
Select observations
   ↓
.loc / .iloc
   ↓
Boolean logic
   ↓
Filter
   ↓
Validate
   ↓
Analysis population
```

Next week we will extend these ideas to **joining, merging, concatenating, reshaping, and missing-data strategies**.
