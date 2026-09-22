# 8.1 Activity

## Ethics, Fairness, Accountability & Transparency

**Tools:** Python, pandas, NumPy, matplotlib/seaborn, Jupyter/VS Code

## Learning objectives

By the end of this tutorial, you should be able to:

- identify several forms of bias in a data workflow;
- evaluate group representation;
- calculate group-specific classification metrics;
- compute demographic parity, true-positive rates, and false-positive rates;
- investigate threshold tradeoffs;
- quantify uncertainty from small subgroup samples;
- identify proxy and measurement concerns;
- write a compact model card;
- document intended and out-of-scope uses;
- design a basic monitoring plan;
- distinguish technical performance from responsible deployment.

---

# Part 0 — Set up

Create:

```text
notebooks/week15_ethics.ipynb
```

Import:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```

Create:

```python
rng = np.random.default_rng(
    556
)
```

---

# Part 1 — Create a synthetic screening dataset

We will use a fictional screening system.

Create:

```python
n = 1000

group = rng.choice(
    ["A", "B"],
    size=n,
    p=[
        0.7,
        0.3
    ]
)

score = rng.normal(
    0,
    1,
    size=n
)

group_shift = np.where(
    group == "A",
    0.2,
    -0.2
)

latent = (
    score
    + group_shift
    + rng.normal(
        0,
        0.8,
        size=n
    )
)

y_true = (
    latent > 0
).astype(int)

predicted_probability = (
    1
    / (
        1
        + np.exp(
            -(
                1.4 * score
                + 0.2
            )
        )
    )
)

df = pd.DataFrame({
    "group": group,
    "score": score,
    "y_true": y_true,
    "probability": predicted_probability
})
```

Inspect:

```python
df.head()
df.shape
```

### Important

This dataset is synthetic.

The purpose is to study fairness concepts without making claims about any real population.

---

# Part 2 — Representation

Calculate:

```python
df["group"].value_counts()
```

and:

```python
df["group"].value_counts(
    normalize=True
)
```

### Questions

1. Are the groups equally represented?
2. Is unequal representation automatically unfair?
3. What additional context would you need?

---

# Part 3 — Outcome prevalence

Calculate:

```python
pd.crosstab(
    df["group"],
    df["y_true"],
    normalize="index"
)
```

### Question

Do base rates differ across groups?

Why could that matter when interpreting fairness metrics?

---

# Part 4 — Apply a classification threshold

Set:

```python
THRESHOLD = 0.5
```

Create:

```python
df["y_pred"] = (
    df["probability"]
    >= THRESHOLD
).astype(int)
```

Inspect:

```python
pd.crosstab(
    df["group"],
    df["y_pred"],
    normalize="index"
)
```

---

# Part 5 — Overall accuracy

Calculate:

```python
accuracy = (
    df["y_pred"]
    == df["y_true"]
).mean()

accuracy
```

### Question

Does this value tell you whether performance is similar across groups?

---

# Part 6 — Group-specific accuracy

Calculate:

```python
group_accuracy = (
    df.assign(
        correct=(
            df["y_pred"]
            == df["y_true"]
        )
    )
    .groupby(
        "group"
    )["correct"]
    .mean()
)

group_accuracy
```

### Reflection

How does this change your understanding of model performance?

---

# Part 7 — Confusion matrix components

Write:

```python
def confusion_counts(
    data
):
    tp = (
        (
            data["y_true"] == 1
        )
        & (
            data["y_pred"] == 1
        )
    ).sum()

    tn = (
        (
            data["y_true"] == 0
        )
        & (
            data["y_pred"] == 0
        )
    ).sum()

    fp = (
        (
            data["y_true"] == 0
        )
        & (
            data["y_pred"] == 1
        )
    ).sum()

    fn = (
        (
            data["y_true"] == 1
        )
        & (
            data["y_pred"] == 0
        )
    ).sum()

    return {
        "tp": tp,
        "tn": tn,
        "fp": fp,
        "fn": fn
    }
```

Test on each group.

---

# Part 8 — Define fairness-relevant metrics

Write:

```python
def classification_metrics(
    data
):
    c = confusion_counts(
        data
    )

    tpr = (
        c["tp"]
        / (
            c["tp"]
            + c["fn"]
        )
    )

    fpr = (
        c["fp"]
        / (
            c["fp"]
            + c["tn"]
        )
    )

    ppv = (
        c["tp"]
        / (
            c["tp"]
            + c["fp"]
        )
    )

    positive_rate = (
        data["y_pred"]
        .mean()
    )

    return {
        "tpr": tpr,
        "fpr": fpr,
        "ppv": ppv,
        "positive_rate": positive_rate
    }
```

---

# Part 9 — Calculate group metrics

Run:

```python
metrics = {}

for group_name, subset in df.groupby(
    "group"
):
    metrics[
        group_name
    ] = classification_metrics(
        subset
    )

pd.DataFrame(
    metrics
).T
```

Interpret each column.

---

# Part 10 — Demographic parity difference

Extract positive prediction rates.

Calculate:

```python
dp_difference = (
    metrics["A"][
        "positive_rate"
    ]
    - metrics["B"][
        "positive_rate"
    ]
)

dp_difference
```

### Question

What would a value near zero mean?

Does that prove the classifier is fair?

---

# Part 11 — Equal opportunity difference

Calculate:

```python
tpr_difference = (
    metrics["A"][
        "tpr"
    ]
    - metrics["B"][
        "tpr"
    ]
)

tpr_difference
```

Interpret:

> Among truly positive cases, how differently are groups treated?

---

# Part 12 — False-positive-rate difference

Calculate:

```python
fpr_difference = (
    metrics["A"][
        "fpr"
    ]
    - metrics["B"][
        "fpr"
    ]
)

fpr_difference
```

### Question

Why might this matter even if true-positive rates are equal?

---

# Part 13 — Predictive parity comparison

Compare:

```python
metrics["A"]["ppv"]
```

with:

```python
metrics["B"]["ppv"]
```

### Question

What does PPV answer that TPR does not?

---

# Part 14 — Metric table

Create a DataFrame containing:

```text
group
n
base_rate
positive_rate
accuracy
TPR
FPR
PPV
```

Round only for display.

Do not round intermediate calculations.

---

# Part 15 — Visualize the disparities

Create a long-form metric table.

Plot:

```python
sns.barplot(
    data=metric_long,
    x="metric",
    y="value",
    hue="group"
)
```

### Reflection

Which differences are most visually substantial?

---

# Part 16 — Sample size beside metrics

Add group counts to your metric report.

### Question

Why should a fairness metric never be interpreted without knowing how many observations support it?

---

# Part 17 — Bootstrap uncertainty for TPR

Use Week 11's bootstrap logic.

For each group:

1. select the group's rows;
2. bootstrap rows with replacement;
3. calculate TPR for each bootstrap sample;
4. calculate a 95% percentile interval.

### Important

Handle bootstrap samples where a denominator becomes zero.

### Reflection

How much uncertainty is there around each group's TPR?

---

# Part 18 — Small subgroup experiment

Create a smaller subgroup:

```python
small_b = (
    df.loc[
        df["group"] == "B"
    ]
    .sample(
        n=30,
        random_state=556
    )
)
```

Calculate TPR and bootstrap interval.

### Question

How did uncertainty change?

---

# Part 19 — Threshold sweep

Create:

```python
thresholds = np.linspace(
    0.1,
    0.9,
    17
)
```

For each threshold:

1. generate predictions;
2. calculate TPR by group;
3. calculate FPR by group;
4. calculate positive rate by group.

Store results.

---

# Part 20 — Plot threshold tradeoffs

Plot:

```text
threshold vs. TPR
threshold vs. FPR
threshold vs. positive rate
```

by group.

### Questions

1. Does one threshold optimize every metric?
2. What tradeoffs become visible?
3. Which decisions are technical, and which are policy choices?

---

# Part 21 — Compare overall and subgroup optimization

Find the threshold with highest overall accuracy.

Then inspect group-specific metrics at that threshold.

### Reflection

Does maximizing aggregate accuracy guarantee acceptable subgroup performance?

---

# Part 22 — Calibration by group

Create probability bins:

```python
df["prob_bin"] = pd.cut(
    df["probability"],
    bins=np.linspace(
        0,
        1,
        6
    ),
    include_lowest=True
)
```

Calculate by:

```text
group
probability bin
```

the:

```text
mean predicted probability
observed positive rate
count
```

Plot predicted vs. observed.

---

# Part 23 — Calibration interpretation

### Question

If both groups are calibrated, does that guarantee:

```text
equal TPR?
equal FPR?
equal positive rates?
```

Explain why fairness definitions are not interchangeable.

---

# Part 24 — Proxy-variable thought experiment

Suppose the dataset includes:

```text
region_code
school_id
neighborhood_score
```

and group membership is removed.

Discuss:

1. Could these variables still contain group information?
2. Would removing `group` make auditing easier or harder?
3. Why is "fairness through unawareness" incomplete?

No code is required.

---

# Part 25 — Measurement-bias thought experiment

Suppose the outcome variable is:

```text
prior system approval
```

rather than:

```text
true future success
```

### Questions

1. What exactly would the model learn?
2. Could historical bias enter the label?
3. Would excellent predictive accuracy solve this problem?

---

# Part 26 — Missingness audit

Introduce:

```python
audit = df.copy()

missing_index = (
    audit.loc[
        audit["group"] == "B"
    ]
    .sample(
        frac=0.2,
        random_state=556
    )
    .index
)

audit.loc[
    missing_index,
    "score"
] = np.nan
```

Calculate missingness by group.

### Question

What process might produce unequal missingness in a real system?

---

# Part 27 — Data minimization exercise

Your hypothetical data source contains:

```text
name
email
date_of_birth
ZIP code
group
score
outcome
device ID
full browsing history
```

The model requires only a subset.

Write:

```text
Keep:
Do not keep:
Need justification:
```

for each field.

### Reflection

Why is data minimization an ethical and technical principle?

---

# Part 28 — Intended use

Write a short description:

```markdown
## Intended use

This model is intended to...
```

Include:

- who should use it;
- what decision it supports;
- what population it was evaluated on.

---

# Part 29 — Out-of-scope use

Write:

```markdown
## Out-of-scope use
```

Give at least three examples of uses that are not supported by the analysis.

### Question

Why is documenting misuse risk important?

---

# Part 30 — Performance section

Create a model-card table with:

```text
overall accuracy
group-specific accuracy
TPR by group
FPR by group
PPV by group
positive rate by group
sample size by group
```

Include uncertainty where possible.

---

# Part 31 — Limitations section

Write at least five limitations.

Possible categories:

```text
synthetic data
sample representativeness
measurement validity
small subgroup uncertainty
threshold dependence
distribution shift
proxy variables
```

Be specific.

---

# Part 32 — Accountability section

Write:

```markdown
## Accountability

Model owner:
Decision owner:
Monitoring owner:
Escalation path:
Appeal/review mechanism:
```

### Reflection

Why should these roles be explicit before deployment?

---

# Part 33 — Transparency section

Write:

```markdown
## Transparency
```

Describe:

- what the model outputs;
- what information users should see;
- what limitations should be communicated;
- what should not be implied.

---

# Part 34 — Monitoring plan

Create:

```text
Metric
Frequency
Population/subgroup
Alert condition
Owner
Action
```

Include at least:

```text
overall performance
TPR/FPR by group
missingness
calibration
data drift
```

---

# Part 35 — Distribution-shift simulation

Create a future dataset where:

```python
future_score = rng.normal(
    0.5,
    1.2,
    size=n
)
```

Apply the same model formula and threshold.

Compare current vs. future:

```text
probability distribution
accuracy
positive rate
group metrics
```

### Question

Why can a previously validated model become inappropriate?

---

# Part 36 — Feedback-loop discussion

Consider:

```text
model labels individuals high risk
      ↓
they receive more scrutiny
      ↓
more negative events are recorded
      ↓
future training data contain more negatives
```

Answer:

1. Is the future label independent of model deployment?
2. What monitoring could reveal this?
3. How might this affect retraining?

---

# Part 37 — NIST RMF mapping exercise

Use the four functions:

```text
Govern
Map
Measure
Manage
```

For your fictional screening system, write one concrete action for each.

Example structure:

```text
Govern:
Map:
Measure:
Manage:
```

---

# Part 38 — Responsible deployment decision

Choose one:

```text
Deploy
Deploy with safeguards
Do not deploy yet
```

Write a one-paragraph justification based on:

- technical performance;
- subgroup metrics;
- uncertainty;
- limitations;
- accountability;
- monitoring readiness.

There is no automatically correct answer.

The reasoning matters.

---

# Part 39 — Final ethics audit

Answer:

```text
What problem is the system solving?
Who is affected?
What assumptions are being made?
What data limitations exist?
What groups may experience different errors?
Which fairness metric matters most here, and why?
What metric conflicts exist?
How uncertain are subgroup estimates?
What should users be told?
Who is accountable?
How will harm be detected after deployment?
What would trigger suspension or retraining?
```

---

# Part 40 — Git checkpoint

Run:

```bash
git status
```

Commit:

```bash
git add .
git commit -m "Complete Week 15 responsible data science exercises"
git push
```

---

# Part 41 — Final course reflection

Answer in Markdown.

### 1. Technical correctness

Why is correct code not sufficient for responsible data science?

### 2. Bias

What is the difference among selection, historical, and measurement bias?

### 3. Fairness

Why is there no single universally correct fairness metric?

### 4. Error rates

Why must false positives and false negatives be interpreted in context?

### 5. Transparency

What information should users know about a model?

### 6. Accountability

Why should responsibility remain with people and organizations rather than "the algorithm"?

### 7. Documentation

How do model cards and data documentation improve responsible practice?

### 8. Monitoring

Why must responsible evaluation continue after deployment?

### 9. Reproducibility

How does reproducibility support accountability?

### 10. STA 556

Which technical practice from this course most improves your ability to build trustworthy statistical workflows?

---

# Completion checklist

- [ ] Created Week 15 notebook
- [ ] Generated a synthetic classification dataset
- [ ] Audited group representation
- [ ] Compared base rates
- [ ] Applied a classification threshold
- [ ] Calculated overall accuracy
- [ ] Calculated subgroup accuracy
- [ ] Built confusion-matrix counts
- [ ] Calculated TPR
- [ ] Calculated FPR
- [ ] Calculated PPV
- [ ] Calculated positive prediction rate
- [ ] Calculated demographic-parity difference
- [ ] Calculated equal-opportunity difference
- [ ] Compared false-positive rates
- [ ] Compared predictive parity
- [ ] Built a group metric table
- [ ] Visualized subgroup metrics
- [ ] Reported subgroup sample sizes
- [ ] Bootstrapped uncertainty for a fairness metric
- [ ] Investigated a small subgroup
- [ ] Swept classification thresholds
- [ ] Visualized threshold tradeoffs
- [ ] Compared overall vs. subgroup optimization
- [ ] Audited calibration by group
- [ ] Discussed proxy variables
- [ ] Discussed label/measurement bias
- [ ] Audited missingness by group
- [ ] Completed a data-minimization exercise
- [ ] Wrote intended-use documentation
- [ ] Wrote out-of-scope-use documentation
- [ ] Created a model-card performance section
- [ ] Documented limitations
- [ ] Defined accountability roles
- [ ] Wrote a transparency section
- [ ] Designed a monitoring plan
- [ ] Simulated distribution shift
- [ ] Discussed feedback loops
- [ ] Mapped the project to Govern/Map/Measure/Manage
- [ ] Made a responsible deployment recommendation
- [ ] Completed the final ethics audit
- [ ] Committed Week 15 work to Git
- [ ] Pushed work to GitHub

---

# What you should now understand

```text
Technically correct workflow
      ↓
Ethical problem definition
      ↓
Representative + appropriate data
      ↓
Performance + subgroup evaluation
      ↓
Fairness tradeoffs
      ↓
Transparency + documentation
      ↓
Accountability
      ↓
Monitoring
      ↓
Responsible statistical practice
```

This completes STA 556: from computational foundations and data engineering through reproducibility, numerical methods, testing, performance, and responsible data science.
