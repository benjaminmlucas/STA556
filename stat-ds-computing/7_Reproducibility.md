# 4.2 Reproducibility & Reporting

## Why this week matters

In Week 7, we began turning repeated analysis steps into reusable functions and modules. This week asks how we turn those computational components into documents that another person can **read, rerun, verify, and regenerate**.

The central question is:

> **How do we make a statistical analysis reproducible from data and code all the way to the final report?**

The approved STA 556 schedule identifies Week 8 as **Reproducibility & Reporting**, including:

- literate programming;
- Quarto;
- Jupyter;
- automated reports;
- dynamic documents.

The goal is not simply to produce an attractive report. It is to create a document whose results are **generated from the same code that performs the analysis**.

---

# 1. What is reproducibility?

A computational result is reproducible when another person can use the same:

```text
data
+
code
+
computational environment
+
instructions
```

to regenerate the same result.

A reproducible workflow therefore requires more than saving a notebook.

Conceptually:

```text
Raw data
   ↓
Code
   ↓
Environment
   ↓
Analysis
   ↓
Figures / tables
   ↓
Report
```

Every arrow matters.

---

# 2. Reproducibility vs. replicability

These terms are used differently across disciplines, but a useful distinction is:

### Reproducibility

Can the same analysis be rerun using the same data and code?

### Replicability

Can an independent study or analysis obtain consistent conclusions using new data or independently implemented methods?

STA 556 focuses mainly on **computational reproducibility**.

---

# 3. Why reproducibility matters

Without a reproducible workflow, common questions become difficult:

- Which version of the data produced this figure?
- Which code produced Table 2?
- What packages were installed?
- What filtering rules were used?
- Was the notebook executed from top to bottom?
- Were any values copied manually into the report?
- Can the report be regenerated six months later?

Reproducibility creates an audit trail from source data to reported result.

---

# 4. The copy-and-paste problem

Suppose you calculate:

```python
mean_score = df["score"].mean()
```

and manually type:

```text
The mean score was 84.7.
```

into a Word document.

If the data change, the code may now return:

```text
85.3
```

but the report still says:

```text
84.7
```

This is a synchronization failure.

A dynamic report instead derives the reported value directly from the computation.

---

# 5. Literate programming

**Literate programming** combines:

```text
narrative
+
code
+
results
```

in a single source document.

Instead of separating:

```text
analysis.py
results.txt
figure.png
report.docx
```

we create a computational document that explains the analysis and executes the code that produces the results.

The idea is:

> **The analysis and the explanation should live close enough together that they cannot easily drift apart.**

---

# 6. Jupyter notebooks

A Jupyter notebook contains:

```text
Markdown cells
Code cells
Outputs
```

Example structure:

```text
# Research question

Narrative explanation

[code cell]
load data

## Descriptive statistics

[code cell]
calculate summary

## Visualization

[code cell]
create figure

## Interpretation

Narrative explanation
```

This makes notebooks useful for exploratory and explanatory analysis.

---

# 7. The hidden-state problem

Jupyter notebooks are interactive.

That is useful—but it creates a reproducibility risk.

Suppose cells are executed in this order:

```text
Cell 1
Cell 5
Cell 3
Cell 8
```

The displayed notebook may no longer represent the actual computation sequence.

A notebook can appear correct while depending on hidden state.

---

# 8. Restart and run all

A basic notebook reproducibility test is:

```text
Restart kernel
      ↓
Run all cells from top to bottom
      ↓
Does the notebook complete successfully?
```

If not, the notebook is not self-contained.

This should become a routine habit before submission.

---

# 9. Execution order should match reading order

Prefer:

```text
imports
   ↓
configuration
   ↓
load data
   ↓
clean data
   ↓
analysis
   ↓
visualization
   ↓
interpretation
```

Avoid code that requires readers to jump backward and rerun arbitrary cells.

A reproducible notebook should behave like a program even though it is interactive.

---

# 10. Keep reusable logic outside the notebook

Week 7 introduced modules.

Instead of placing every function in the notebook:

```python
def clean_data(...):
    ...

def summarize(...):
    ...

def plot_results(...):
    ...
```

we can use:

```python
from src.analysis import (
    clean_data,
    summarize,
    plot_results
)
```

This separates:

```text
reusable computational logic
```

from:

```text
analysis narrative
```

---

# 11. Quarto

Quarto is an open-source scientific and technical publishing system.

It supports:

- Python;
- Jupyter;
- Markdown;
- equations;
- citations;
- cross-references;
- figures;
- tables;
- HTML;
- PDF;
- Word;
- presentations;
- websites.

For Python users, Quarto can execute embedded Python code and regenerate the document from source.

---

# 12. A basic Quarto document

A `.qmd` file may begin with:

```yaml
---
title: "STA 556 Analysis Report"
author: "Student Name"
format: html
jupyter: python3
---
```

Then Markdown:

```markdown
# Introduction

This report analyzes...
```

and Python:

````markdown
```{python}
import pandas as pd

df = pd.read_csv("data/processed/study.csv")
```
````

---

# 13. Rendering

From the terminal:

```bash
quarto render report.qmd
```

Quarto runs the computation and creates the requested output.

For HTML:

```bash
quarto render report.qmd --to html
```

For Word:

```bash
quarto render report.qmd --to docx
```

For PDF:

```bash
quarto render report.qmd --to pdf
```

Quarto's Python integration supports executable code blocks and can regenerate computational output when documents are rendered. citeturn600654search0turn600654search3

---

# 14. Preview vs. render

During development:

```bash
quarto preview report.qmd
```

In GitHub Codespaces, Quarto runs the preview server **inside the Codespace**, not on your laptop. Codespaces detects the preview server and exposes it through a forwarded port. Use the **Open in Browser** notification or the VS Code **Ports** panel rather than manually typing a `localhost` address.

provides a live preview.

For the final output:

```bash
quarto render report.qmd
```

A useful workflow is:

```text
edit
 ↓
preview
 ↓
revise
 ↓
render
```

---

# 15. Quarto and Jupyter are complementary

You do not necessarily need to choose between them.

Possible workflows include:

```text
.ipynb
  ↓
Quarto render
  ↓
HTML / PDF / Word
```

or:

```text
.qmd
  ↓
Jupyter executes Python
  ↓
HTML / PDF / Word
```

Quarto can render Jupyter notebooks directly, and can execute them when requested. citeturn600654search0turn600654search2

---

# 16. Dynamic values

Suppose:

```python
mean_score = df["score"].mean()
```

A dynamic report can insert that calculated quantity rather than requiring a manual copy-and-paste step.

The broader principle is:

> **If a number can be computed, it should usually be generated from code rather than manually transcribed.**

---

# 17. Dynamic figures

A report should generate figures from code:

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots()

ax.hist(df["score"])
ax.set_xlabel("Score")
ax.set_ylabel("Frequency")

plt.show()
```

Rather than manually exporting a figure, editing it elsewhere, and inserting it into the report.

The source document should contain the logic that creates the figure.

---

# 18. Dynamic tables

Tables can also be computed:

```python
summary = (
    df.groupby("group")
    ["score"]
    .agg(["count", "mean", "std"])
)

summary
```

If the source data change, the table changes automatically on rerender.

---

# 19. Code visibility

Not every audience needs to see every code block.

Quarto can control whether code is visible.

For example:

```yaml
execute:
  echo: false
```

or per block:

````markdown
```{python}
#| echo: false

summary = df.describe()
summary
```
````

This allows the same computational workflow to serve technical and nontechnical audiences.

---

# 20. Code folding

For HTML documents, code can be included but folded.

This gives readers the option to inspect computational details without forcing every code block into the main narrative.

This is useful when:

- reproducibility matters;
- code is relevant;
- but the primary audience wants the results first.

---

# 21. Separate warnings and messages from results

Computational output may include:

```text
warnings
messages
printed diagnostics
tables
figures
```

A final report should deliberately control what is displayed.

Development information can be useful during analysis but distracting in the finished report.

---

# 22. Figures need captions

A reproducible report should treat figures as document elements rather than screenshots.

A useful figure includes:

```text
figure
caption
number / reference
```

For example:

```markdown
Figure 1 shows the distribution of assessment scores.
```

Cross-references reduce errors when figures are reordered.

---

# 23. Tables need context

Avoid dropping an unexplained DataFrame into a report.

Instead:

```text
Table 1 summarizes participant characteristics by group.
```

Then show a table with:

- meaningful labels;
- appropriate rounding;
- units;
- missingness where relevant.

The statistical interpretation belongs in prose.

---

# 24. Reproducible environments

Code may fail on another machine because the environment differs.

For example:

```text
Python 3.12
pandas 2.x
numpy 2.x
matplotlib
seaborn
```

A reproducible project should record dependencies.

One simple method:

```bash
python -m pip freeze > requirements.txt
```

Quarto's documentation explicitly recommends saving package requirements so that an environment can be recreated later. citeturn600654search5

---

# 25. Restore an environment

Given:

```text
requirements.txt
```

another user can install dependencies:

```bash
python -m pip install -r requirements.txt
```

This command documents how an environment can be recreated, but **students normally do not need to run it in STA 556 Codespaces** because the course dependencies are installed automatically when the Codespace is created. Treat `requirements.txt` as part of the reproducibility record rather than as a weekly installation step.

This does not solve every reproducibility issue, but it records an important part of the computational environment.

---

# 26. Randomness must be controlled

Suppose:

```python
import numpy as np

x = np.random.normal(
    size=100
)
```

Each execution can produce different values.

For reproducibility:

```python
rng = np.random.default_rng(556)

x = rng.normal(
    size=100
)
```

A random seed makes stochastic computations repeatable.

This will become especially important during the simulation modules later in STA 556.

---

# 27. Raw data should remain unchanged

A reproducible workflow usually looks like:

```text
data/raw/
     ↓
cleaning code
     ↓
data/processed/
     ↓
analysis
     ↓
report
```

Avoid:

```text
open raw CSV
manually edit cells
save over original
```

Manual changes are difficult to audit.

---

# 28. Avoid manual report edits

Suppose Quarto generates:

```text
report.docx
```

If you manually change statistical values in the rendered document, the source no longer represents the report.

A useful principle is:

> **Treat rendered outputs as products, not as the source of truth.**

Edit:

```text
.qmd
.ipynb
.py
```

then rerender.

---

# 29. One source, multiple outputs

A powerful feature of reproducible publishing is:

```text
one source
   ├── HTML
   ├── PDF
   └── Word
```

Quarto supports many output formats from the same computational source. citeturn600654search4

This reduces duplication and keeps outputs synchronized.

---

# 30. Parameterized reports

Suppose one report must be produced for each region:

```text
Arizona
California
Nevada
Utah
```

Instead of copying the report four times, define a parameter such as:

```text
state
```

Then reuse the same analysis logic.

Conceptually:

```text
report template
      +
parameter
      ↓
specific report
```

This is the beginning of automated reporting.

---

# 31. Automated reporting

A reproducible report can become part of a pipeline:

```text
new data
   ↓
ingestion
   ↓
cleaning
   ↓
analysis
   ↓
render report
```

For example:

```bash
python src/prepare_data.py
quarto render report.qmd
```

Eventually these commands can be automated.

The essential idea is that human copy-and-paste work is replaced by deterministic computation.

---

# 32. Cache carefully

Some computations are expensive.

Quarto supports caching and stored computation options. citeturn600654search0

Caching can reduce unnecessary recomputation, but introduces a question:

> **Is the cached result still valid for the current data and code?**

Caching improves efficiency only when invalidation is understood.

---

# 33. Reproducibility checklist

A report should ideally answer:

### Data

- Where did the data come from?
- Which version was used?
- Are raw data preserved?

### Code

- Is all transformation code available?
- Is reusable code modularized?
- Are random seeds controlled?

### Environment

- What Python/package versions are required?

### Execution

- Can the analysis run from beginning to end?

### Report

- Are figures and tables generated from code?
- Are values dynamically computed?
- Can the document be regenerated?

---

# 34. Reproducible does not automatically mean correct

A workflow can be perfectly reproducible and still reproduce the wrong answer.

For example:

```python
incorrect_model(data)
```

may run identically every time.

Therefore:

```text
reproducibility ≠ validity
```

We still need:

- correct statistical reasoning;
- validated data;
- appropriate methods;
- testing;
- critical interpretation.

Reproducibility makes the workflow inspectable.

---

# 35. A strong project structure

For STA 556 Codespaces, use a Quarto project configuration at the repository root when reports live in a subdirectory:

```yaml
project:
  type: default
  execute-dir: project
```

This makes Quarto execute code relative to the project root, matching the course convention for `data/`, `src/`, and other repository-relative paths.


A Week 8 project might look like:

```text
project/
├── README.md
├── requirements.txt
├── data/
│   ├── raw/
│   └── processed/
├── notebooks/
├── src/
│   └── analysis.py
├── reports/
│   └── week08_report.qmd
└── figures/
```

The structure communicates the computational workflow.

---

# 36. Key ideas

By the end of Week 8, you should be able to explain:

1. What computational reproducibility means.
2. Why copy-and-paste reporting creates errors.
3. What literate programming is.
4. Strengths and risks of Jupyter notebooks.
5. Why restart-and-run-all matters.
6. How modular functions support reproducibility.
7. What Quarto does.
8. How `.qmd` documents combine Markdown and executable Python.
9. The difference between previewing and rendering.
10. How Jupyter and Quarto can work together.
11. Why dynamic values, figures, and tables are preferable to manual transcription.
12. Why dependencies should be recorded.
13. Why random seeds matter.
14. Why raw data should remain unchanged.
15. Why rendered reports should not become the source of truth.
16. How one source can generate several output formats.
17. What parameterized and automated reporting mean.
18. Why reproducibility is necessary but not sufficient for statistical correctness.

---

# 37. Recommended reading

## Quarto — Using Python

Official documentation for executable Python code, Jupyter integration, rendering, execution, and caching. citeturn600654search0turn600654search3

https://quarto.org/docs/computations/python

## Quarto — Jupyter Computations Tutorial

A practical tutorial on controlling code, output, and rendered computational documents. citeturn600654search1

https://quarto.org/docs/get-started/computations/jupyter.html

## Quarto — Jupyter Authoring Tutorial

Covers document formats, equations, citations, cross-references, and other authoring tools. citeturn600654search2

https://quarto.org/docs/get-started/authoring/jupyter.html

## Quarto — Virtual Environments

Useful for connecting reproducible reports to reproducible Python environments. citeturn600654search5

https://quarto.org/docs/projects/virtual-environments.html

---

# 38. YouTube recommendations

## 1. Data Umbrella — "Reproducible Publications with Python and Quarto" — Thomas Mock

A particularly strong fit for STA 556. The talk explains Quarto as a reproducible publishing system and demonstrates Python/Jupyter integration, executable documents, multi-format publishing, parameters, and reproducible computational publications. citeturn600654youtube35

**Recommended use:** Watch the first 30–40 minutes alongside the theory material. The sections on computational documents, rendering, Jupyter integration, and parameters map closely onto Week 8.

[Watch on YouTube](https://www.youtube.com/watch?v=TnVgHE9LAiw)

---

## 2. Posit / Keith Galli — "Quarto Crash Course | Create Professional Reports, Dashboards & Websites w/ Markdown & Python Code!"

A practical Python-based Quarto walkthrough covering setup, Markdown, HTML/PDF/Word outputs, parameters, and automated report generation. citeturn600654youtube36

**Recommended use:** Use selected chapters during or after the hands-on tutorial. The installation/setup, basic reporting, static document, and automated-report sections are particularly relevant.

[Watch on YouTube](https://www.youtube.com/watch?v=_VKxTPWDhA4)

---

## 3. Jupyter reproducibility / restart-and-run-all

For additional reinforcement, find a short Jupyter tutorial emphasizing kernel state, execution order, and restarting/running notebooks from top to bottom.

**Recommended use:** Optional reinforcement for students who regularly work interactively in notebooks.

[Find Jupyter reproducibility videos on YouTube](https://www.youtube.com/results?search_query=Jupyter+notebook+reproducibility+restart+run+all)

---

# 39. Week 8 takeaway

The central lesson is:

> **A report should be the output of the analysis workflow—not a manually assembled description of it.**

The progression is:

```text
Raw data
   ↓
Reproducible environment
   ↓
Modular code
   ↓
Computational document
   ↓
Dynamic values / tables / figures
   ↓
Render
   ↓
HTML / PDF / Word
   ↓
Reproduce
```

Next week we move into **Matrix Computation & Linear Algebra**, including vectorization, broadcasting, and matrix decompositions for statistical computing.
