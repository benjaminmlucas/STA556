# 8.1 Ethics, Fairness, Accountability & Transparency

## Why this week matters

Throughout STA 556, we have emphasized reproducibility, validation, numerical correctness, and efficient computation.

But technically correct code can still produce harmful or misleading results.

A workflow can be:

```text
reproducible
fast
well-tested
statistically sophisticated
```

and still be ethically poor if:

- the data are unrepresentative;
- the outcome is badly defined;
- sensitive groups are affected differently;
- model limitations are hidden;
- users misunderstand what the system can do;
- there is no meaningful accountability for errors.

The central question for Week 15 is:

> **How do we evaluate whether a data-science workflow is responsible, not merely technically correct?**

This week focuses on:

- data ethics;
- bias and representativeness;
- fairness metrics;
- measurement and label bias;
- privacy and consent;
- transparency;
- explainability;
- accountability;
- documentation;
- model/data cards;
- tradeoffs among competing goals;
- responsible deployment and monitoring.

---

# 1. Ethics is not an add-on

Ethics should not appear only after the analysis is finished.

A responsible workflow considers ethical questions throughout:

```text
problem definition
      ↓
data collection
      ↓
measurement
      ↓
modeling
      ↓
evaluation
      ↓
deployment
      ↓
monitoring
```

Ethical issues are often created early and cannot be fixed later by changing a metric.

---

# 2. Start with the problem definition

Before collecting data, ask:

```text
What decision is being supported?
Who benefits?
Who bears risk?
What happens if the model is wrong?
Is prediction actually needed?
```

A technically elegant model may solve the wrong problem.

---

# 3. Stakeholders

A stakeholder is anyone affected by the system.

Possible stakeholders include:

```text
model developers
decision makers
data subjects
customers
patients
students
employees
communities
regulators
```

The people most affected are not always the people building the model.

---

# 4. Data are not neutral

A dataset is created through choices.

Examples:

```text
who was sampled
who was excluded
what was measured
how variables were coded
what counts as missing
which outcome was chosen
when data were collected
```

These decisions shape what the model can learn.

---

# 5. Selection bias

Suppose the target population is:

```text
all eligible individuals
```

but the observed dataset contains:

```text
only people who previously entered the system
```

Then the sample may not represent the population of interest.

A larger sample does not automatically solve selection bias.

---

# 6. Historical bias

A model trained on historical decisions may reproduce historical patterns.

Suppose the target label is:

```text
previously approved
```

The model may learn:

```text
how decisions were historically made
```

rather than:

```text
what an ideal future decision should be
```

Labels are not always ground truth.

---

# 7. Measurement bias

A variable may measure different constructs across groups.

Examples:

```text
healthcare utilization ≠ healthcare need
arrest records ≠ underlying offending
test score ≠ complete ability
clicks ≠ satisfaction
```

A mathematically precise model can still optimize a poor proxy.

---

# 8. Proxy variables

A feature may indirectly encode sensitive information.

For example:

```text
ZIP code
school attended
language
purchase history
```

may correlate with demographic characteristics.

Removing a sensitive variable does not guarantee that a model is insensitive to group membership.

---

# 9. Missingness can be ethical information

Missing data may reflect:

```text
unequal access
different measurement processes
administrative burden
systematic nonresponse
```

Week 4 emphasized:

> **Missingness is data.**

In Week 15, we add:

> **Missingness may also reveal how a system treats different groups.**

---

# 10. Privacy

Data science often uses information people did not expect to be combined or reused.

Privacy questions include:

```text
Was consent meaningful?
Is the data necessary?
Who can access it?
How long is it retained?
Can individuals be re-identified?
Could outputs reveal sensitive information?
```

Collecting more data is not automatically better.

---

# 11. Data minimization

A useful principle is:

> **Collect and retain only what is genuinely needed for the task.**

Benefits include:

- lower privacy risk;
- simpler governance;
- lower storage cost;
- less exposure if data are leaked.

---

# 12. Fairness is not one metric

There is no universal scalar quantity called:

```text
fairness
```

Different fairness definitions represent different values and goals.

Examples include:

```text
demographic parity
equal opportunity
equalized odds
predictive parity
calibration
```

These definitions can conflict.

---

# 13. Group fairness notation

For binary classification, let:

```text
Y = true outcome
Ŷ = predicted decision
A = group membership
```

We may compare outcomes across groups.

The choice of metric depends on the harm being considered.

---

# 14. Demographic parity

Demographic parity asks whether:

```text
P(Ŷ = 1 | A=a)
```

is similar across groups.

This compares positive prediction rates.

It does not require those positive predictions to be correct.

---

# 15. Equal opportunity

Equal opportunity asks whether true-positive rates are similar:

```text
P(Ŷ = 1 | Y=1, A=a)
```

across groups.

Interpretation:

> Among people who truly qualify, are groups equally likely to receive a positive prediction?

---

# 16. Equalized odds

Equalized odds considers both:

```text
true-positive rate
false-positive rate
```

across groups.

This asks whether model error rates are comparable conditional on the true outcome.

---

# 17. Predictive parity

Predictive parity considers:

```text
P(Y=1 | Ŷ=1, A=a)
```

across groups.

This is related to positive predictive value.

Interpretation:

> Among people receiving a positive prediction, is the probability of actually being positive similar across groups?

---

# 18. Calibration

A model is calibrated within groups if:

```text
predicted probability ≈ observed frequency
```

within each group.

For example:

```text
among people assigned risk 0.7
about 70% experience the outcome
```

Calibration is useful but does not imply equal error rates.

---

# 19. Fairness metrics can conflict

When groups have different base rates, several desirable fairness criteria may be impossible to satisfy simultaneously.

Therefore:

> **Fairness is not solved by selecting every metric and demanding they all be equal.**

The metric must match the decision context and relevant harm.

---

# 20. False positives and false negatives are not symmetric

In many applications:

```text
false positive cost
≠
false negative cost
```

Examples:

```text
wrongly denying access
missing a dangerous condition
incorrectly flagging fraud
failing to detect fraud
```

Fairness analysis should consider the meaning of each error.

---

# 21. Aggregate accuracy can hide disparities

Suppose overall accuracy is:

```text
92%
```

But group-level accuracy is:

```text
Group A: 97%
Group B: 71%
```

The overall number hides important variation.

Always ask whether important performance metrics differ across relevant subgroups.

---

# 22. Intersectionality

Performance may be acceptable for:

```text
group A
group B
```

when evaluated separately, but poor for an intersection such as:

```text
A + age category 3
```

Aggregated group analysis can hide small but highly affected subgroups.

However, very small subgroup sizes also create statistical uncertainty.

---

# 23. Sample size matters in fairness analysis

A difference in metrics may reflect:

```text
real disparity
sampling variability
small subgroup size
```

Fairness metrics should be accompanied by:

- counts;
- uncertainty intervals where appropriate;
- contextual interpretation.

---

# 24. Fairness through unawareness is insufficient

A common idea is:

```text
remove sensitive attribute
```

and assume the model is fair.

But correlated proxy variables may still encode the same information.

Furthermore, if group membership is never recorded, it may be impossible to audit disparities.

---

# 25. Transparency

Transparency means making relevant information visible.

Examples:

```text
what the model was designed for
what data were used
what variables are included
how performance was measured
known limitations
who is responsible
```

Transparency does not mean exposing every line of code to every user.

---

# 26. Explainability vs. transparency

These ideas are related but different.

### Transparency

Information about:

```text
system design
data
process
limitations
governance
```

### Explainability

Information about:

```text
why a particular prediction or decision occurred
```

A transparent system is not automatically interpretable, and an interpretable system is not automatically fair.

NIST's AI Risk Management Framework treats transparency, accountability, explainability, privacy, validity, safety, and fairness as distinct but interacting characteristics of trustworthy systems.

---

# 27. Accountability

Accountability answers:

```text
Who is responsible for what happens?
```

Questions include:

```text
Who approves deployment?
Who monitors performance?
Who responds to failures?
Can users appeal?
Who owns the final decision?
```

A system should not become an excuse for saying:

> "The model decided."

---

# 28. Human-in-the-loop is not automatically responsible

Adding a human reviewer does not guarantee accountability.

Possible problems:

```text
automation bias
rubber-stamping
insufficient time
poor training
unclear authority
```

A human must have meaningful ability and responsibility to intervene.

---

# 29. Documentation

Documentation is part of responsible data science.

Useful artifacts include:

```text
data documentation
model cards
decision logs
assumption lists
risk assessments
change logs
```

Documentation supports transparency and future auditing.

---

# 30. Model cards

A model card can document:

```text
intended use
out-of-scope use
training/evaluation data
performance
subgroup performance
ethical considerations
limitations
monitoring requirements
```

The goal is not bureaucracy.

The goal is to prevent critical assumptions from living only in the developer's memory.

---

# 31. Data documentation

For datasets, document:

```text
source
collection process
population
sampling method
variables
missingness
known biases
license/consent
transformations
limitations
```

This extends Week 5's provenance ideas.

---

# 32. Accountability through reproducibility

Reproducibility supports accountability because it makes it possible to ask:

```text
Which data created this result?
Which code produced it?
Which version was deployed?
Which threshold was used?
```

But reproducibility is only one component of responsibility.

---

# 33. Thresholds are policy choices

A probabilistic classifier may output:

```text
0.73
```

The final decision depends on a threshold:

```text
predict positive if probability >= 0.50
```

Changing the threshold changes:

```text
TPR
FPR
precision
positive decision rate
```

Threshold choice is not purely technical.

It encodes tradeoffs among errors.

---

# 34. Fairness and thresholds

Different thresholds across groups may improve one fairness metric while violating another principle.

There is no automatic technical answer.

The appropriate policy depends on:

```text
legal context
ethical goals
costs of errors
stakeholder values
deployment setting
```

Data scientists should surface these tradeoffs rather than silently choose them.

---

# 35. Distribution shift

A model evaluated successfully at deployment may degrade later.

Reasons include:

```text
population change
behavior change
policy change
measurement change
feedback loops
```

Responsible deployment requires monitoring.

---

# 36. Feedback loops

Model predictions can change the world that generates future data.

Example pattern:

```text
model predicts high risk
      ↓
more monitoring
      ↓
more events observed
      ↓
future data show more events
      ↓
model reinforces original pattern
```

This is a feedback loop.

Observational data after deployment may no longer represent an independent environment.

---

# 37. Monitor more than accuracy

Deployment monitoring can include:

```text
overall performance
subgroup performance
drift
missingness
calibration
false-positive rates
false-negative rates
appeals/complaints
unexpected use
```

Technical metrics should be paired with operational evidence.

---

# 38. NIST AI Risk Management Framework

The NIST AI RMF is a voluntary framework for managing AI risks.

Its core functions are:

```text
Govern
Map
Measure
Manage
```

The framework emphasizes trustworthy characteristics including:

```text
valid and reliable
safe
secure and resilient
accountable and transparent
explainable and interpretable
privacy-enhanced
fair with harmful bias managed
```

This is a useful framework for thinking beyond a single fairness metric.

---

# 39. Ethics is socio-technical

A model is part of a larger system:

```text
data
people
institutions
policies
software
incentives
decision processes
```

Therefore:

> **Ethical evaluation must examine the system around the model, not only the algorithm.**

---

# 40. Technical tradeoffs should be explicit

Suppose a change:

```text
reduces false negatives
```

but:

```text
increases false positives
```

The data scientist should not hide that tradeoff behind a single accuracy score.

Responsible communication makes the tradeoff explicit.

---

# 41. Uncertainty belongs in ethical communication

Do not report:

```text
Group B TPR = 0.71
```

as if it were exact when the subgroup has only 14 positive observations.

Communicate uncertainty and sample size.

Ethical communication includes honest representation of what the data support.

---

# 42. Responsible visualization

Visualization can mislead through:

```text
truncated axes
selective groups
omitted uncertainty
poor color choices
missing denominators
```

Week 6's visualization principles are also ethical principles.

---

# 43. Responsible optimization

Week 12 showed that an optimizer faithfully minimizes the objective provided.

If the objective is poorly chosen:

```text
the optimizer can efficiently optimize the wrong thing
```

Mathematical optimization does not provide ethical justification for the target.

---

# 44. Responsible performance engineering

Week 14 emphasized speed.

But performance improvements can create ethical tradeoffs if they:

```text
remove auditability
reduce interpretability
use approximations without disclosure
discard minority cases as "outliers"
```

Fast code is not inherently better code.

---

# 45. A responsible data-science checklist

Before deployment, ask:

### Problem

- Is prediction needed?
- What decision will be made?
- Who is affected?

### Data

- Is the sample representative?
- Are variables valid measures?
- Is missingness systematic?
- Are sensitive proxies present?
- Was data use appropriate?

### Model

- What objective is optimized?
- What are the main error types?
- Are subgroup metrics examined?
- Is uncertainty reported?

### Deployment

- Who is accountable?
- Can decisions be challenged?
- How is performance monitored?
- What happens when the model fails?

### Documentation

- Are intended uses documented?
- Are limitations explicit?
- Can results be reproduced?

---

# 46. Key ideas

By the end of Week 15, you should be able to explain:

1. Why technically correct analysis can still be ethically poor.
2. How problem definition affects downstream ethics.
3. Selection, historical, and measurement bias.
4. Proxy variables.
5. Privacy and data minimization.
6. Why fairness is not a single metric.
7. Demographic parity.
8. Equal opportunity.
9. Equalized odds.
10. Predictive parity and calibration.
11. Why fairness metrics can conflict.
12. Why subgroup sample size and uncertainty matter.
13. Transparency vs. explainability.
14. Accountability.
15. Why human oversight is not automatically sufficient.
16. Model/data documentation.
17. Threshold choice as a policy decision.
18. Distribution shift and feedback loops.
19. Deployment monitoring.
20. Why responsible data science is socio-technical.

---

# 47. Recommended reading

## NIST — AI Risk Management Framework

A voluntary framework for incorporating trustworthiness and risk management into the design, development, deployment, and evaluation of AI systems.

https://www.nist.gov/itl/ai-risk-management-framework

## NIST — AI RMF Playbook

Suggested actions organized around:

```text
Govern
Map
Measure
Manage
```

https://www.nist.gov/itl/ai-risk-management-framework/nist-ai-rmf-playbook

## NIST AI Resource Center

Resources for testing, evaluation, verification, validation, and responsible AI practice.

https://airc.nist.gov/

## Model Cards for Model Reporting

Mitchell et al. introduced model cards as a structured way to document model uses, limitations, evaluation, and ethical considerations.

https://arxiv.org/abs/1810.03993

## Datasheets for Datasets

Gebru et al. proposed structured documentation for datasets, including motivation, composition, collection, preprocessing, distribution, and maintenance.

https://arxiv.org/abs/1803.09010

---

# 48. YouTube recommendations

## 1. NIST — AI Risk Management Framework Explainer

NIST provides official explanatory resources for the AI RMF, which is a useful way to connect technical evaluation to broader governance, transparency, accountability, and risk management.

**Recommended use:** Watch before the governance/accountability portion of the week.

[Find NIST AI RMF videos on YouTube](https://www.youtube.com/results?search_query=NIST+AI+Risk+Management+Framework)

---

## 2. Fairness in Machine Learning

A focused introduction to fairness metrics is useful before the hands-on group-metric calculations.

**Recommended use:** Watch before the demographic-parity, equal-opportunity, and equalized-odds exercises.

[Find fairness-in-ML tutorials on YouTube](https://www.youtube.com/results?search_query=machine+learning+fairness+demographic+parity+equal+opportunity+equalized+odds)

---

## 3. Model Cards and Responsible AI Documentation

A short model-card tutorial provides practical context for the documentation exercise in this week's tutorial.

**Recommended use:** Optional preparation for the model-card activity.

[Find model-card tutorials on YouTube](https://www.youtube.com/results?search_query=model+cards+responsible+AI+tutorial)

---

# 49. Week 15 takeaway

The central lesson is:

> **Responsible data science requires more than accurate models: it requires careful problem definition, representative data, explicit tradeoffs, transparent documentation, meaningful accountability, and ongoing monitoring.**

```text
Problem definition
      ↓
Data + measurement
      ↓
Model + objective
      ↓
Performance + fairness
      ↓
Transparency
      ↓
Accountability
      ↓
Deployment
      ↓
Monitoring
      ↓
Responsible revision
```

This completes STA 556's progression from computational tools to responsible statistical practice.
