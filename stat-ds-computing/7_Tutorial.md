# 4.2 Activity

## Reproducibility & Reporting with Jupyter and Quarto

**Tools:** Python, Jupyter, Quarto, pandas, matplotlib, Git

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

- evaluate whether a notebook is reproducible;
- eliminate hidden execution-state dependencies;
- create a computational report;
- combine Markdown, Python, tables, and figures;
- render a Quarto document;
- generate HTML and other outputs from one source;
- dynamically compute reported quantities;
- import reusable functions from `src/`;
- record Python dependencies;
- control randomness;
- separate raw, processed, and report outputs;
- perform a final reproducibility test.

---

# Part 0 — Project setup

Extend your existing STA 556 project:

```text
project/
├── README.md
├── data/
│   ├── raw/
│   └── processed/
├── notebooks/
├── src/
├── figures/
├── reports/
└── tests/
```

Create:

```text
reports/week08_report.qmd
```

Also create:

```text
notebooks/week08_reproducibility.ipynb
```

Create a small Quarto project configuration at the **repository root**:

```text
_quarto.yml
```

with:

```yaml
project:
  type: default
  execute-dir: project
```

`execute-dir: project` tells Quarto to execute report code from the project root. This keeps paths such as `data/processed/week08_data.csv` and imports such as `from src...` consistent with the rest of the course.

---

# Part 1 — Verify the provided computational environment

From the terminal:

```bash
python --version
```

Check:

```bash
quarto --version
```

If `quarto --version` fails in the official course Codespace, treat that as a course-environment issue rather than installing Quarto manually.

Then:

```bash
python -m pip show pandas
```

and:

```bash
python -m pip show matplotlib
```

### Reflection

Why could package versions matter if the same source code is run several years later?

---

# Part 2 — Create a deliberately fragile notebook

In:

```text
notebooks/week08_reproducibility.ipynb
```

create cells in this order.

### Cell 1

```python
import pandas as pd
import numpy as np
```

### Cell 2

```python
mean_score
```

### Cell 3

```python
rng = np.random.default_rng(556)

df = pd.DataFrame({
    "group": rng.choice(
        ["A", "B"],
        size=50
    ),
    "score": rng.normal(
        80,
        10,
        size=50
    )
})
```

### Cell 4

```python
mean_score = df["score"].mean()
```

Now execute:

```text
Cell 1
Cell 3
Cell 4
Cell 2
```

The notebook appears to work.

---

# Part 3 — Restart and run all

Restart the kernel.

Then run all cells from top to bottom.

### Question

What fails?

Why?

Fix the notebook by moving computations before anything that depends on them.

Repeat:

```text
Restart kernel
Run all
```

until the notebook succeeds.

---

# Part 4 — Add a reproducible random generator

Compare:

```python
x = np.random.normal(
    size=5
)
```

with:

```python
rng = np.random.default_rng(556)

x = rng.normal(
    size=5
)
```

Restart the kernel and rerun.

### Question

Which one reproduces the same values?

Why will this matter during future simulation modules?

---

# Part 5 — Build a small processed dataset

Create:

```python
rng = np.random.default_rng(556)

n = 100

df = pd.DataFrame({
    "id": range(1, n + 1),
    "group": rng.choice(
        ["A", "B"],
        size=n
    ),
    "age": rng.normal(
        35,
        9,
        size=n
    ),
    "score": rng.normal(
        80,
        10,
        size=n
    )
})
```

Then:

```python
df["age"] = df["age"].round(1)
```

Save:

```python
df.to_csv(
    "data/processed/week08_data.csv",
    index=False
)
```

---

# Part 6 — Create a reusable summary function

In:

```text
src/week08_analysis.py
```

write:

```python
import pandas as pd
```

Then:

```python
def summarize_groups(
    data: pd.DataFrame
) -> pd.DataFrame:
    return (
        data.groupby(
            "group"
        )["score"]
        .agg(
            n="count",
            mean="mean",
            sd="std"
        )
        .reset_index()
    )
```

Save.

---

# Part 7 — Use your module

In the notebook:

```python
from src.week08_analysis import (
    summarize_groups
)
```

Then:

```python
summary = summarize_groups(df)
summary
```

### Reflection

Why is this preferable to copying the summary logic into every report?

---

# Part 8 — Create a basic Quarto document

Open:

```text
reports/week08_report.qmd
```

Add:

```yaml
---
title: "STA 556 Week 8 Reproducible Report"
author: "Your Name"
format: html
jupyter: python3
---
```

Then:

```markdown
# Introduction

This report demonstrates a reproducible statistical
workflow using Python and Quarto.
```

---

# Part 9 — Add setup code

Add a Python block:

````markdown
```{python}
import pandas as pd
import matplotlib.pyplot as plt

from src.week08_analysis import summarize_groups
```
````

Depending on where your report is located, you may need to ensure the project root is the working directory.

### Question

Why should path assumptions be explicit?

---

# Part 10 — Load the data

Add:

````markdown
```{python}
df = pd.read_csv(
    "data/processed/week08_data.csv"
)

df.head()
```
````

### Important

Do not manually recreate the DataFrame inside the report.

The report should consume the processed analytical data.

---

# Part 11 — Add a data-validation block

Add:

````markdown
```{python}
assert len(df) == 100
assert df["id"].is_unique
assert df["score"].notna().all()
```
````

### Question

Why include checks in a report-generation workflow?

What happens if the upstream data unexpectedly change?

---

# Part 12 — Generate a summary table

Add:

````markdown
```{python}
summary = summarize_groups(df)
summary
```
````

Add narrative:

```markdown
## Descriptive statistics

The table below summarizes assessment scores by group.
```

### Reflection

If the data change, what must you manually update?

Ideally: nothing.

---

# Part 13 — Generate a dynamic value

Calculate:

```python
overall_mean = df["score"].mean()
```

Use the result in your narrative.

If your chosen Quarto/Jupyter workflow supports inline execution, insert it dynamically.

If not, generate a compact results object in a code block and reference the reported value immediately below it.

### Principle

> Do not manually copy a statistic if the document can generate it.

---

# Part 14 — Create a figure in the report

Add:

````markdown
```{python}
fig, ax = plt.subplots(
    figsize=(7, 5)
)

ax.hist(
    df["score"],
    bins=15
)

ax.set(
    title="Distribution of assessment scores",
    xlabel="Assessment score",
    ylabel="Frequency"
)

plt.show()
```
````

Render later and verify the figure appears.

---

# Part 15 — Add a figure caption

Use Quarto cell options:

````markdown
```{python}
#| label: fig-score-distribution
#| fig-cap: "Distribution of assessment scores."

fig, ax = plt.subplots()

ax.hist(
    df["score"],
    bins=15
)

ax.set(
    xlabel="Assessment score",
    ylabel="Frequency"
)

plt.show()
```
````

Now reference the figure in your prose according to Quarto's cross-reference syntax.

---

# Part 16 — Render the report

From the project root:

```bash
quarto render reports/week08_report.qmd
```

Locate the resulting HTML file.

Open it.

### Questions

1. Did the Python code execute?
2. Did the table appear?
3. Did the figure appear?
4. Did the narrative render correctly?

---

# Part 17 — Preview during development

Run:

```bash
quarto preview reports/week08_report.qmd
```

In GitHub Codespaces, Quarto runs the preview server **inside the Codespace**, not on your laptop. Codespaces detects the preview server and exposes it through a forwarded port. Use the **Open in Browser** notification or the VS Code **Ports** panel rather than manually typing a `localhost` address.

Edit the report and observe the preview update.

### Reflection

When is `preview` useful?

When would you use `render` instead?

---

# Part 18 — Control code visibility

At the top of the document, experiment with:

```yaml
execute:
  echo: false
```

Render.

What changed?

Now remove it and instead hide only one code block:

````markdown
```{python}
#| echo: false

summary
```
````

### Question

Why might a report hide code from one audience but show it to another?

---

# Part 19 — Code folding

For HTML, experiment with:

```yaml
format:
  html:
    code-fold: true
```

Render.

### Reflection

How does code folding balance readability and transparency?

---

# Part 20 — Generate a Word version

Change:

```yaml
format:
  html: default
  docx: default
```

Then:

```bash
quarto render reports/week08_report.qmd
```

Inspect both outputs.

### Question

What benefit comes from generating both from one source?

---

# Part 21 — Optional PDF output

If your environment has the required PDF tooling, add:

```yaml
pdf: default
```

and render.

If PDF rendering fails because a TeX dependency is missing, document the dependency rather than treating it as an analysis error.

---

# Part 22 — Add meaningful labels

Review all figures and tables.

Replace:

```text
score
age
group
```

with reader-friendly labels where appropriate:

```text
Assessment score
Age (years)
Study group
```

### Principle

The report is for a reader, not for the DataFrame.

---

# Part 23 — Build a proper report structure

Organize:

```markdown
# Introduction

# Data

# Methods

# Results

## Descriptive statistics

## Visualization

# Discussion
```

### Challenge

Write 1–3 sentences for each section.

Keep the report concise.

---

# Part 24 — Separate computation from interpretation

Example:

```markdown
## Results

[computed table]

Group A had a higher mean score than Group B.
```

The table is computational output.

The sentence is statistical interpretation.

### Question

Why should these remain conceptually distinct?

---

# Part 25 — Make the report fail deliberately

Rename:

```text
data/processed/week08_data.csv
```

temporarily.

Render the report.

### Question

What happens?

Why is an obvious failure preferable to silently using old manually copied results?

Restore the file afterward.

---

# Part 26 — Record the environment

From the terminal:

```bash
python -m pip freeze > requirements.txt
```

Open:

```text
requirements.txt
```

Inspect several entries.

### Question

What does this file record?

What does it not record?

---

# Part 27 — Reproducibility test in a fresh environment

Conceptually, another user should be able to run:

```bash
python -m pip install -r requirements.txt
```

This command documents how an environment can be recreated, but **students normally do not need to run it in STA 556 Codespaces** because the course dependencies are installed automatically when the Codespace is created. Treat `requirements.txt` as part of the reproducibility record rather than as a weekly installation step.

and then:

```bash
quarto render reports/week08_report.qmd
```

You do not need to destroy your current environment.

Instead, explain the steps another researcher would follow.

---

# Part 28 — Make a clean execution test

Before the final render:

1. restart the Jupyter kernel;
2. run the notebook from beginning to end;
3. save it;
4. render the Quarto report;
5. inspect warnings/errors;
6. verify the reported numbers.

### Question

Why is this stronger than simply checking whether the final HTML file exists?

---

# Part 29 — Remove manual values

Search your report for manually typed statistical quantities.

For example:

```text
The mean score was 81.37.
```

Ask:

> Could this value be generated computationally?

If yes, replace the manual workflow.

---

# Part 30 — One-source-many-outputs challenge

Configure the report to produce at least:

```text
HTML
DOCX
```

from the same `.qmd`.

Render both.

Compare:

- tables;
- figures;
- headings;
- code;
- overall readability.

### Reflection

What formatting differences exist despite sharing one source?

---

# Part 31 — Parameterized-report concept

Imagine you need one report per group.

Instead of:

```text
report_A.qmd
report_B.qmd
```

think:

```text
report.qmd
+
group parameter
```

Sketch how your functions would need to work:

```python
def prepare_group_report(
    data,
    group
):
    ...
```

### Question

What advantages does this have over copying and editing reports?

---

# Part 32 — Automation concept

Create a small shell script conceptually containing:

```bash
python src/prepare_data.py
quarto render reports/week08_report.qmd
```

You do not need sophisticated automation yet.

The point is to see the report as the final step of a computational pipeline.

---

# Part 33 — Cache discussion

Suppose your analysis takes two hours.

Would you want Quarto to recompute it every time you change one sentence?

Discuss:

- benefits of caching;
- risk of stale computation;
- when recomputation is preferable.

---

# Part 34 — Audit the report

Use this checklist.

## Data

- Is the data source clear?
- Is the processed file generated reproducibly?
- Are raw data preserved?

## Code

- Are transformations in code?
- Are reusable functions in `src/`?
- Are assumptions checked?

## Environment

- Is `requirements.txt` present?

## Randomness

- Are seeds controlled?

## Report

- Are statistics generated dynamically?
- Are figures generated from code?
- Are tables generated from code?
- Can the report render from beginning to end?

---

# Part 35 — Peer reproducibility test

If possible, exchange repositories with a classmate.

Without verbal instructions, attempt to determine:

1. Which file is the report source?
2. Which data file does it use?
3. Which Python module does it import?
4. How should the report be rendered?
5. What dependencies are required?

### Important

If your partner must ask you where everything is, improve the project documentation.

---

# Part 36 — Update the README

Add:

```markdown
## Week 8 report

To render the report:

```bash
quarto render reports/week08_report.qmd
```

Required Python dependencies are listed in:

```text
requirements.txt
```
```

Also document where input data are located.

---

# Part 37 — Git checkpoint

Run:

```bash
git status
```

Review carefully.

Do you want to commit:

```text
.qmd source?
requirements.txt?
src module?
processed data?
rendered HTML?
rendered Word document?
```

Your choice should be deliberate.

Then:

```bash
git add .
git commit -m "Complete Week 8 reproducible reporting workflow"
git push
```

---

# Part 38 — Final reflection

Answer in Markdown.

### 1. Reproducibility

What information is required to reproduce a computational analysis?

### 2. Jupyter

What is hidden state, and why can it be dangerous?

### 3. Literate programming

What does it mean to combine narrative and computation?

### 4. Quarto

What role does Quarto play in the workflow?

### 5. Dynamic reporting

Why are dynamically generated statistics preferable to manually copied values?

### 6. Environments

Why should package dependencies be recorded?

### 7. Randomness

Why should random seeds be explicit?

### 8. Rendered outputs

Why should rendered Word/HTML/PDF files usually not become the primary source document?

### 9. Modularity

How did the Week 7 function/module work improve this week's report?

### 10. Scientific validity

Why can a workflow be reproducible but still statistically wrong?

---

# Completion checklist

- [ ] Created Week 8 notebook
- [ ] Created a Quarto `.qmd` report
- [ ] Demonstrated hidden notebook state
- [ ] Restarted and ran a notebook from top to bottom
- [ ] Controlled randomness with a generator/seed
- [ ] Created processed data
- [ ] Created a reusable analysis module
- [ ] Imported module functions into the workflow
- [ ] Added Quarto YAML metadata
- [ ] Loaded data from the report
- [ ] Added validation checks
- [ ] Generated a dynamic summary table
- [ ] Generated a figure from code
- [ ] Added a figure caption
- [ ] Rendered an HTML report
- [ ] Used `quarto preview`
- [ ] Controlled code visibility
- [ ] Experimented with code folding
- [ ] Generated a Word report
- [ ] Explored PDF rendering if available
- [ ] Added reader-friendly labels
- [ ] Structured the report into narrative sections
- [ ] Deliberately triggered a reproducibility failure
- [ ] Created `requirements.txt`
- [ ] Documented how to restore dependencies
- [ ] Performed a clean execution test
- [ ] Removed manually copied statistical values
- [ ] Produced multiple output formats from one source
- [ ] Explored parameterized-report design
- [ ] Explored automated rendering
- [ ] Audited the project for reproducibility
- [ ] Attempted a peer reproducibility test
- [ ] Updated the README
- [ ] Committed Week 8 work to Git
- [ ] Pushed work to GitHub

---

# What you should now understand

```text
Raw data
   ↓
Documented environment
   ↓
Modular code
   ↓
Computational document
   ↓
Execute from beginning to end
   ↓
Generate tables + figures + values
   ↓
Render
   ↓
HTML / Word / PDF
   ↓
Another person can reproduce it
```

Next week we move into **Matrix Computation & Linear Algebra**, including broadcasting, vectorized operations, SVD, and QR decomposition for statistical computing.
