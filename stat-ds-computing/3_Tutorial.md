# 2.1 Activity

## DataFrames, Indexing, Slicing & Filtering

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

- create and inspect pandas DataFrames;
- distinguish Series from DataFrames;
- select columns;
- select rows using `.loc` and `.iloc`;
- slice rows and columns;
- create Boolean conditions;
- filter with one or several conditions;
- use `.isin()`, `.isna()`, and `.notna()`;
- modify selected observations with `.loc`;
- translate statistical inclusion criteria into reproducible code;
- validate a filtered dataset.

---

# Part 0 — Set up your Week 3 work

Use the project/repository you have been building during Weeks 1 and 2.

Create:

```text
notebooks/week03_dataframes.ipynb
```

At the top:

```python
import pandas as pd
```

---

# Part 1 — Create a DataFrame

Create:

```python
data = {
    "id": [101, 102, 103, 104, 105, 106],
    "age": [24, 31, 27, 19, 42, 35],
    "group": ["A", "B", "A", "B", "A", "B"],
    "score": [81, 94, 88, 72, 91, 85]
}

df = pd.DataFrame(data)
```

Display:

```python
df
```

Then inspect:

```python
df.head()
df.shape
df.columns
df.index
df.dtypes
df.info()
df.describe()
```

### Questions

1. How many observations are there?
2. How many variables?
3. Which columns are numeric?
4. What is the default index?
5. What does `df.shape` return?

---

# Part 2 — Series vs. DataFrame

Run:

```python
age = df["age"]
type(age)
```

Now:

```python
age_df = df[["age"]]
type(age_df)
```

### Question

Explain the difference between:

```python
df["age"]
```

and:

```python
df[["age"]]
```

---

# Part 3 — Select columns

Select:

```python
df["score"]
```

Then:

```python
df[["age", "score"]]
```

### Challenge

Create `analysis_df` containing only:

- `id`
- `group`
- `score`

---

# Part 4 — Select rows by position

Use:

```python
df.iloc[0]
```

Then:

```python
df.iloc[0:3]
```

Then:

```python
df.iloc[[0, 2, 5]]
```

Now select rows 1–3 and columns 1–2:

```python
df.iloc[1:4, 1:3]
```

### Questions

1. What does the first argument to `.iloc[]` control?
2. What does the second argument control?
3. Is the stop value included in positional slicing?

---

# Part 5 — Create a meaningful index

Set:

```python
df = df.set_index("id")
```

Inspect:

```python
df
df.index
```

Compare:

```python
df.loc[101]
```

with:

```python
df.iloc[0]
```

### Reflection

They identify the same observation here. Why do the two commands mean different things?

---

# Part 6 — Select rows by label

Try:

```python
df.loc[101]
```

```python
df.loc[[101, 103, 105]]
```

Then:

```python
df.loc[
    [101, 103],
    ["age", "score"]
]
```

### Challenge

Select IDs 102, 104, and 106 and display only:

```text
age
group
score
```

---

# Part 7 — `.loc` vs. `.iloc`

Create:

```python
small = pd.DataFrame(
    {"value": [10, 20, 30, 40]},
    index=[101, 205, 310, 412]
)
```

Compare:

```python
small.loc[205]
```

with:

```python
small.iloc[1]
```

Complete:

```text
.loc  uses ______________________
.iloc uses ______________________
```

---

# Part 8 — Slicing

Try:

```python
df.iloc[1:4]
```

Then:

```python
df.loc[102:105]
```

Count the returned rows.

### Question

Why does label-based slicing behave differently from positional slicing?

---

# Part 9 — Select rows and columns together

Try:

```python
df.loc[:, ["age", "score"]]
```

Then:

```python
df.loc[
    102:105,
    ["age", "score"]
]
```

### Challenge

Select IDs 101 through 104 and show only `group` and `score`.

---

# Part 10 — Create a Boolean condition

Run:

```python
condition = df["score"] > 80
```

Inspect:

```python
condition
type(condition)
```

Now:

```python
df.loc[condition]
```

### Question

What does each `True` and `False` represent?

---

# Part 11 — Simple filters

Find participants:

```python
df.loc[df["score"] > 80]
```

Then:

```python
df.loc[df["age"] >= 30]
```

Then:

```python
df.loc[df["group"] == "A"]
```

### Challenge

Create separate subsets for participants who are:

- younger than 30;
- in group B;
- scoring at least 85.

---

# Part 12 — Multiple conditions

Use AND:

```python
df.loc[
    (df["age"] < 30) &
    (df["score"] > 80)
]
```

Use OR:

```python
df.loc[
    (df["group"] == "A") |
    (df["score"] > 90)
]
```

Use NOT:

```python
df.loc[
    ~(df["group"] == "A")
]
```

### Challenge

Find participants who are at least 30 years old **and** score at least 85.

---

# Part 13 — Predict before running

Without executing the code, predict which IDs will appear:

```python
df.loc[
    (df["group"] == "A") &
    (df["score"] >= 90)
]
```

Write down your prediction and then run the code.

### Question

Did your result match your prediction?

---

# Part 14 — Filter rows and choose variables

Try:

```python
df.loc[
    df["score"] >= 85,
    ["age", "group", "score"]
]
```

### Challenge

Return only `group` and `score` for participants aged at least 30.

---

# Part 15 — `.isin()`

Create:

```python
df2 = pd.DataFrame(
    {
        "id": range(201, 209),
        "group": ["A", "B", "C", "A", "C", "B", "A", "C"],
        "score": [81, 94, 88, 72, 91, 85, 79, 96]
    }
).set_index("id")
```

Select groups A and C:

```python
df2.loc[
    df2["group"].isin(["A", "C"])
]
```

### Challenge

Select observations in groups B or C **and** with score at least 85.

---

# Part 16 — Missing values

Create:

```python
missing_df = pd.DataFrame(
    {
        "id": [1, 2, 3, 4, 5],
        "age": [21, 32, None, 45, 28],
        "score": [81, None, 77, 92, 88]
    }
).set_index("id")
```

Inspect:

```python
missing_df
missing_df.isna()
missing_df.isna().sum()
```

Find missing scores:

```python
missing_df.loc[
    missing_df["score"].isna()
]
```

Find observed scores:

```python
missing_df.loc[
    missing_df["score"].notna()
]
```

### Reflection

Why is `score = 0` different from `score = missing` in statistical analysis?

---

# Part 17 — Assign values with `.loc`

Return to `df`.

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

Then:

```python
df.loc[
    df["score"] < 80,
    "performance"
] = "low"
```

### Challenge

Create an `age_group` variable:

```text
young → age < 30
older → age >= 30
```

using `.loc`.

---

# Part 18 — Avoid chained assignment

Consider:

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

### Question

Why is the `.loc` version clearer? Connect your answer to Week 2's discussion of references and mutation.

---

# Part 19 — Define an analysis population

Suppose the study protocol says:

> Include adults in group A who have a non-missing score.

Write:

```python
analysis = df.loc[
    (df["age"] >= 18) &
    (df["group"] == "A") &
    (df["score"].notna())
]
```

### Reflection

Explain the code in ordinary statistical language.

Then complete:

```text
D* = {i : ______________________________}
```

---

# Part 20 — Write criteria first

Suppose your analysis question is:

> What are the scores among older high-performing participants?

Before writing code, define:

```text
Age criterion:
Score criterion:
Missing-data criterion:
Variables required:
```

Then translate the rules into pandas:

```python
eligible = (
    (df["age"] >= 30) &
    (df["score"] >= 85) &
    (df["score"].notna())
)

analysis_df = df.loc[
    eligible,
    ["age", "group", "score"]
]
```

---

# Part 21 — Mini data-analysis challenge

Create:

```python
participants = pd.DataFrame(
    {
        "id": range(1001, 1011),
        "age": [22, 34, 29, 41, 19, 37, 26, 45, 31, 28],
        "group": ["A", "B", "A", "C", "B", "A", "C", "B", "A", "C"],
        "score": [78, 91, 85, 88, 72, 95, 81, 67, 90, 84]
    }
).set_index("id")
```

Complete each task.

## A. Participants aged 30 or older

```python
older = ...
```

## B. Participants in groups A or C

```python
groups_ac = ...
```

## C. Participants with scores at least 85

```python
high_scores = ...
```

## D. Participants aged at least 30 AND scoring at least 85

```python
older_high = ...
```

## E. Only group and score for high scorers

```python
high_score_summary = ...
```

## F. Group A participants scoring at least 80

```python
group_a_good = ...
```

## G. Participants who are NOT in group B

```python
not_group_b = ...
```

---

# Part 22 — Verify every answer

For each result, inspect:

```python
result.shape
result.head()
result.index
```

Then manually compare it with the original dataset.

Ask:

> Does every included row satisfy the condition?

and:

> Did I exclude anyone who should be included?

Code that executes successfully is not necessarily correct.

---

# Part 23 — Function challenge

Write a reusable function:

```python
def select_high_scores(
    data: pd.DataFrame,
    minimum_score: float
) -> pd.DataFrame:
    ...
```

It should return observations whose scores are at least `minimum_score`.

Test:

```python
select_high_scores(participants, 85)
```

and:

```python
select_high_scores(participants, 90)
```

### Extension

Modify the function so the caller can also specify which columns to return.

---

# Part 24 — Reproducibility challenge

Imagine you wrote:

```python
good_data = participants.loc[
    participants["score"] > 80
]
```

Six months later, someone asks:

> Why did you choose 80?

Improve the workflow:

```python
MINIMUM_SCORE = 80

good_data = participants.loc[
    participants["score"] > MINIMUM_SCORE
]
```

### Discussion

Does naming the threshold improve reproducibility? What documentation is still missing?

---

# Part 25 — Git checkpoint

Save your work.

```bash
git status
```

Then:

```bash
git add .
```

Commit:

```bash
git commit -m "Complete Week 3 pandas indexing and filtering"
```

Push:

```bash
git push
```

Continue the workflow established in Week 1:

```text
Create
   ↓
Inspect
   ↓
Transform
   ↓
Validate
   ↓
Document
   ↓
Commit
   ↓
Share
```

---

# Part 26 — Final reflection

Answer in Markdown.

### 1. DataFrames

What does a pandas DataFrame provide that a list of dictionaries does not?

### 2. Series

What is the difference between:

```python
df["age"]
```

and:

```python
df[["age"]]
```

?

### 3. `.loc` vs. `.iloc`

Explain the difference in one sentence.

### 4. Boolean filtering

What does:

```python
df["score"] >= 85
```

produce before it is used to subset a DataFrame?

### 5. Analysis populations

Why is a pandas filter more than simply a programming operation?

### 6. Validation

Why should you inspect a filtered dataset even if no Python error occurred?

### 7. Reproducibility

How can explicit filtering criteria make statistical analyses easier to reproduce?

---

# Completion checklist

- [ ] Created the Week 3 notebook
- [ ] Imported pandas
- [ ] Created a DataFrame
- [ ] Used `head()`, `shape`, `dtypes`, `info()`, and `describe()`
- [ ] Distinguished Series and DataFrames
- [ ] Selected individual columns
- [ ] Selected multiple columns
- [ ] Used `.iloc`
- [ ] Used `.loc`
- [ ] Created a meaningful index
- [ ] Compared positional and label selection
- [ ] Practiced slicing
- [ ] Created Boolean conditions
- [ ] Filtered using one condition
- [ ] Filtered using multiple conditions
- [ ] Used `&`, `|`, and `~`
- [ ] Used `.isin()`
- [ ] Used `.isna()` and `.notna()`
- [ ] Assigned values with `.loc`
- [ ] Avoided chained assignment
- [ ] Defined an analysis population
- [ ] Completed the mini data-analysis challenge
- [ ] Wrote a reusable filtering function
- [ ] Validated filtered results
- [ ] Committed Week 3 work to Git
- [ ] Pushed Week 3 work to GitHub

---

# What you should now understand

```text
Raw tabular data
       ↓
DataFrame
       ↓
Inspect
       ↓
Select variables
       ↓
Define conditions
       ↓
Filter observations
       ↓
Validate
       ↓
Analysis-ready subset
```

Next week we will extend the wrangling workflow to **joins, merges, concatenation, reshaping, and missing-data strategies**.
