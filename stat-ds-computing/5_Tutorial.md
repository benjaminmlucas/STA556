# 3.1 Activity
## Exploratory Data Analysis & Visualization

**Estimated time:** 2–3 hours  
**Tools:** Python, pandas, matplotlib, seaborn, plotnine, Jupyter/VS Code

## Learning objectives

By the end of this tutorial, you should be able to:

- use plots as part of exploratory statistical analysis;
- select plots based on variable type;
- create histograms, boxplots, count plots, and scatterplots;
- compare distributions across groups;
- recognize the effect of histogram binning;
- use seaborn for statistical visualization;
- construct plots using Grammar of Graphics concepts;
- distinguish mapped and fixed aesthetics;
- use layers and facets;
- improve labels, scales, and figure design;
- save publication-quality figures;
- identify data-quality problems visually.

---

# Part 0 — Set up

Create:

```text
notebooks/week06_visualization.ipynb
```

Ensure the project contains:

```text
figures/
```

Import:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```

If `plotnine` is installed:

```python
from plotnine import (
    ggplot,
    aes,
    geom_point,
    geom_boxplot,
    geom_jitter,
    geom_smooth,
    facet_wrap,
    labs,
    theme_minimal
)
```

---

# Part 1 — Create an analysis dataset

Use a reproducible random-number generator:

```python
rng = np.random.default_rng(556)
```

Create:

```python
n = 150

df = pd.DataFrame({
    "id": range(1, n + 1),
    "group": rng.choice(
        ["A", "B", "C"],
        size=n
    ),
    "age": rng.normal(
        loc=35,
        scale=10,
        size=n
    ),
    "baseline": rng.normal(
        loc=70,
        scale=12,
        size=n
    )
})
```

Add an outcome with structure:

```python
group_effect = df["group"].map({
    "A": 0,
    "B": 5,
    "C": -4
})

df["score"] = (
    40
    + 0.7 * df["baseline"]
    + 0.25 * df["age"]
    + group_effect
    + rng.normal(0, 8, size=n)
)
```

Round age:

```python
df["age"] = df["age"].round(1)
```

Inspect:

```python
df.head()
df.shape
df.dtypes
df.describe()
```

---

# Part 2 — Identify variable types

Classify each variable:

```text
id:
group:
age:
baseline:
score:
```

Possible labels:

```text
identifier
categorical
continuous
discrete numeric
```

### Question

Why should variable type be considered before choosing a visualization?

---

# Part 3 — One continuous variable

Start numerically:

```python
df["score"].describe()
```

Then visualize:

```python
plt.hist(
    df["score"]
)

plt.show()
```

### Questions

1. Where is the center?
2. Is the distribution symmetric?
3. Are there unusual values?
4. What does the histogram reveal that `describe()` does not?

---

# Part 4 — Investigate binning

Compare:

```python
plt.hist(df["score"], bins=5)
plt.show()
```

```python
plt.hist(df["score"], bins=15)
plt.show()
```

```python
plt.hist(df["score"], bins=40)
plt.show()
```

### Reflection

How does the visual impression change? Is there one objectively correct number of bins?

---

# Part 5 — Density plot

```python
sns.kdeplot(
    data=df,
    x="score"
)

plt.show()
```

Compare the density with the histogram.

### Questions

1. What features are easier to see?
2. What information has been smoothed?
3. Why is this not the raw data?

---

# Part 6 — One categorical variable

```python
df["group"].value_counts()
```

Then:

```python
sns.countplot(
    data=df,
    x="group"
)

plt.show()
```

Calculate proportions:

```python
df["group"].value_counts(
    normalize=True
)
```

Explain the difference between counts and proportions.

---

# Part 7 — Continuous variable by group

```python
sns.boxplot(
    data=df,
    x="group",
    y="score"
)

plt.show()
```

### Questions

1. Which group has the highest median?
2. Which appears most variable?
3. What information is hidden?

---

# Part 8 — Add raw observations

```python
fig, ax = plt.subplots()

sns.boxplot(
    data=df,
    x="group",
    y="score",
    ax=ax
)

sns.stripplot(
    data=df,
    x="group",
    y="score",
    ax=ax
)

plt.show()
```

What becomes visible that was hidden by the boxplot?

---

# Part 9 — Violin plot

```python
sns.violinplot(
    data=df,
    x="group",
    y="score"
)

plt.show()
```

Compare:

```text
boxplot
violin plot
strip plot
```

Which representation best answers which question?

---

# Part 10 — Two continuous variables

```python
sns.scatterplot(
    data=df,
    x="baseline",
    y="score"
)

plt.show()
```

Then:

```python
df[
    ["baseline", "score"]
].corr()
```

### Questions

1. Is the relationship approximately linear?
2. Are there outliers?
3. Does variance appear constant?
4. What does the figure show that correlation does not?

---

# Part 11 — Add a third variable

```python
sns.scatterplot(
    data=df,
    x="baseline",
    y="score",
    hue="group"
)

plt.show()
```

What new statistical question can this figure address?

---

# Part 12 — Faceting

```python
g = sns.relplot(
    data=df,
    x="baseline",
    y="score",
    col="group"
)
```

Compare faceting with color. When are separate panels easier to interpret?

---

# Part 13 — Introduce a data-quality problem

```python
bad_rows = pd.DataFrame({
    "id": [151, 152],
    "group": ["A", "B"],
    "age": [999, 31],
    "baseline": [72, 300],
    "score": [82, 90]
})
```

Combine:

```python
dirty = pd.concat(
    [df, bad_rows],
    ignore_index=True
)
```

Visualize:

```python
sns.histplot(
    data=dirty,
    x="age"
)

plt.show()
```

```python
sns.scatterplot(
    data=dirty,
    x="baseline",
    y="score"
)

plt.show()
```

What problems become obvious visually?

---

# Part 14 — Find problematic observations

```python
dirty.loc[
    dirty["age"] > 120
]
```

```python
dirty.loc[
    dirty["baseline"] > 150
]
```

How does visualization complement explicit validation rules?

---

# Part 15 — Grammar of Graphics

Using `plotnine`:

```python
(
    ggplot(
        df,
        aes(
            x="baseline",
            y="score"
        )
    )
    + geom_point()
)
```

Identify:

```text
data:
x aesthetic:
y aesthetic:
geometry:
```

---

# Part 16 — Map an aesthetic

```python
(
    ggplot(
        df,
        aes(
            x="baseline",
            y="score",
            color="group"
        )
    )
    + geom_point()
)
```

What information does color now encode?

---

# Part 17 — Mapping vs. setting

Compare:

```python
(
    ggplot(
        df,
        aes(
            x="baseline",
            y="score",
            color="group"
        )
    )
    + geom_point()
)
```

with:

```python
(
    ggplot(
        df,
        aes(
            x="baseline",
            y="score"
        )
    )
    + geom_point(
        color="black"
    )
)
```

Explain the difference between mapped and fixed color.

---

# Part 18 — Add a layer

```python
(
    ggplot(
        df,
        aes(
            x="baseline",
            y="score"
        )
    )
    + geom_point()
    + geom_smooth()
)
```

What information does the second geometry add?

---

# Part 19 — Facet in plotnine

```python
(
    ggplot(
        df,
        aes(
            x="baseline",
            y="score"
        )
    )
    + geom_point()
    + facet_wrap("group")
)
```

Compare this with mapping group to color.

---

# Part 20 — Grouped distribution with layers

```python
(
    ggplot(
        df,
        aes(
            x="group",
            y="score"
        )
    )
    + geom_boxplot()
    + geom_jitter(
        width=0.15,
        alpha=0.5
    )
)
```

Which layer provides a summary? Which shows raw observations?

---

# Part 21 — Make a bad plot deliberately

Create a cluttered plot using too many visual encodings such as color, size, and shape.

Then answer:

1. Is more encoded information always better?
2. Which encodings are useful?
3. Which would you remove?

Simplify the plot.

---

# Part 22 — Titles and units

```python
fig, ax = plt.subplots(
    figsize=(7, 5)
)

sns.scatterplot(
    data=df,
    x="age",
    y="score",
    hue="group",
    ax=ax
)

ax.set(
    title="Assessment score by participant age",
    xlabel="Age (years)",
    ylabel="Assessment score"
)

fig.tight_layout()
```

How does this differ from the default exploratory figure?

---

# Part 23 — Axis experiment

Create group means:

```python
group_means = (
    df.groupby(
        "group",
        as_index=False
    )["score"]
    .mean()
)
```

Plot them with a bar chart and experiment with y-axis limits.

### Reflection

How can a truncated baseline exaggerate group differences?

---

# Part 24 — Compare bar and point plots

```python
sns.pointplot(
    data=df,
    x="group",
    y="score"
)

plt.show()
```

If the goal is comparing group means rather than quantities represented by bar lengths, which display is easier to interpret?

---

# Part 25 — Overplotting

```python
large = pd.DataFrame({
    "x": rng.normal(size=5000),
    "y": rng.normal(size=5000)
})
```

Plot:

```python
sns.scatterplot(
    data=large,
    x="x",
    y="y"
)

plt.show()
```

Then:

```python
sns.scatterplot(
    data=large,
    x="x",
    y="y",
    alpha=0.2
)

plt.show()
```

Why does transparency help?

---

# Part 26 — Hexbin alternative

```python
plt.hexbin(
    large["x"],
    large["y"],
    gridsize=30
)

plt.xlabel("x")
plt.ylabel("y")
plt.show()
```

How does this represent dense regions differently?

---

# Part 27 — Visualize missingness

```python
missing = df.copy()

missing.loc[
    missing.sample(
        15,
        random_state=556
    ).index,
    "score"
] = np.nan
```

Create an indicator:

```python
missing["score_missing"] = (
    missing["score"].isna()
)
```

Plot:

```python
sns.countplot(
    data=missing,
    x="group",
    hue="score_missing"
)

plt.show()
```

Does missingness appear equally distributed across groups?

---

# Part 28 — Publication-quality figure

```python
fig, ax = plt.subplots(
    figsize=(7, 5)
)

sns.scatterplot(
    data=df,
    x="baseline",
    y="score",
    hue="group",
    ax=ax
)

ax.set(
    title=(
        "Follow-up score increases "
        "with baseline score"
    ),
    xlabel="Baseline assessment score",
    ylabel="Follow-up assessment score"
)

ax.legend(
    title="Study group"
)

fig.tight_layout()
```

Inspect the figure as a reader rather than as its author.

---

# Part 29 — Save the figure

```python
figure_path = (
    "figures/"
    "week06_baseline_vs_score.png"
)
```

Save:

```python
fig.savefig(
    figure_path,
    dpi=300,
    bbox_inches="tight"
)
```

Also save as PDF:

```python
fig.savefig(
    "figures/"
    "week06_baseline_vs_score.pdf",
    bbox_inches="tight"
)
```

---

# Part 30 — EDA vs. final figure

Create two versions of the same plot.

### Version A — exploratory

Fast and minimally styled.

### Version B — communication

Include:

- descriptive title;
- informative labels;
- units where relevant;
- clear legend;
- suitable figure size;
- intentional output format.

Why is it inefficient to fully polish every exploratory figure?

---

# Part 31 — Visualization design challenge

Choose a visualization for each question before writing code.

### A

What is the distribution of participant age?

### B

How many participants are in each study group?

### C

How does score vary across groups?

### D

Is baseline score associated with follow-up score?

### E

Does that relationship differ by group?

### F

Are there unusual age values?

### G

How does the distribution of score differ among groups?

Explain each choice.

---

# Part 32 — Mini EDA challenge

Using `df`, answer:

1. Which group has the highest typical score?
2. Which group has the greatest variability?
3. Is age related to score?
4. Is baseline related to score?
5. Does the baseline-score relationship appear similar across groups?
6. Are there observations deserving further investigation?

For each include:

```text
statistical question
numerical summary
visualization
brief interpretation
```

---

# Part 33 — Critique your own plot

Choose one figure.

### Statistical

- What question does it answer?
- Are the correct variables represented?
- Is anything important hidden?

### Design

- Is the title meaningful?
- Are axes understandable?
- Is color doing useful work?
- Is the figure cluttered?

### Reproducibility

- Can it be regenerated from code?
- Is it saved predictably?
- Is the source data identifiable?

Revise the figure after the critique.

---

# Part 34 — Git checkpoint

```bash
git status
```

Stage appropriate files:

```bash
git add .
```

Commit:

```bash
git commit -m "Complete Week 6 exploratory visualization"
```

Push:

```bash
git push
```

---

# Part 35 — Final reflection

### 1. EDA

What is the purpose of exploratory data analysis?

### 2. Variable types

Why should variable type influence plot choice?

### 3. Histograms

Why can changing the number of bins change interpretation?

### 4. Grammar of Graphics

What are the roles of data, aesthetics, and geometries?

### 5. Mapping vs. setting

What is the difference?

### 6. Raw data

Why can showing individual observations improve a summary plot?

### 7. Visualization and cleaning

How can plots reveal data-quality problems?

### 8. Communication

What separates an exploratory plot from a publication-quality figure?

### 9. Reproducibility

Why should final figures be created and saved from code rather than manually edited?

---

# Completion checklist

- [ ] Created Week 6 notebook
- [ ] Generated a reproducible analysis dataset
- [ ] Classified variables by type
- [ ] Created histograms
- [ ] Explored histogram binning
- [ ] Created a density plot
- [ ] Created a count plot
- [ ] Created a boxplot
- [ ] Added raw observations to a group comparison
- [ ] Created a violin plot
- [ ] Created a scatterplot
- [ ] Compared scatterplots with correlation
- [ ] Mapped group to a visual aesthetic
- [ ] Used faceting
- [ ] Used visualization to detect bad data
- [ ] Created a plot using plotnine
- [ ] Identified data/aesthetic/geometry components
- [ ] Distinguished mapping from setting
- [ ] Added layers
- [ ] Created faceted Grammar of Graphics plots
- [ ] Simplified an overcomplicated plot
- [ ] Improved titles and labels
- [ ] Investigated axis choices
- [ ] Explored overplotting
- [ ] Used transparency or hexbin aggregation
- [ ] Visualized missingness
- [ ] Created a publication-quality figure
- [ ] Saved PNG and PDF output
- [ ] Completed the mini EDA challenge
- [ ] Critiqued and revised a visualization
- [ ] Committed Week 6 work to Git
- [ ] Pushed work to GitHub

---

# What you should now understand

```text
Statistical question
       ↓
Variable types
       ↓
Numerical summaries
       ↓
Visual encodings
       ↓
Explore
       ↓
Detect structure / problems
       ↓
Revise analysis
       ↓
Communicate clearly
       ↓
Save reproducibly
```

Next week we will move into **Modular Programming & Functions**, including pure functions, variable scope, and functional programming patterns.
