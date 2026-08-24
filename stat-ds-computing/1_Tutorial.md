# 1.1 Activity

## Build Your First Reproducible Python Data Science Project in GitHub Codespaces

**Tools:** GitHub Codespaces, Python, VS Code, Jupyter, Git, GitHub, integrated terminal

## Learning objectives

By the end of this tutorial, you should be able to:

- open and navigate a GitHub Codespace;
- identify the repository root and `/workspaces` location;
- navigate a project using the command line;
- create and run a Python script;
- create and run a Jupyter notebook;
- explain the difference between Git and GitHub;
- inspect repository status and history;
- make commits with meaningful messages;
- push your work to GitHub;
- explain why the Codespace, repository, code, and documentation are all parts of a reproducible workflow.

---

# Part 0 — Open the course Codespace

Before beginning, read:

```text
0_Codespaces_Introduction.md
```

Open the GitHub repository supplied for your STA 556 work and choose **Create codespace** (or reopen your existing Codespace).

When VS Code opens in the browser, open an integrated terminal.

Run:

```bash
pwd
```

You should see a path beginning with:

```text
/workspaces/
```

Now run:

```bash
git status
```

Git should recognize the repository.

### Important

The repository has already been cloned into the Codespace. **Do not run `git init` and do not create another Git repository inside it.**

---

# Part 1 — Check the course environment

Run:

```bash
python --version
git --version
```

Then:

```bash
python -c "import numpy, pandas, matplotlib; print('Core packages available')"
```

### Checkpoint

You should know:

- your current working directory;
- your Python version;
- your Git version;
- that the repository is already under version control;
- that the core Python packages are available.

If a required command or package is missing, do not immediately install it yourself. First check the course instructions or report the environment problem.

---

# Part 2 — Explore the Codespaces interface

Identify the following parts of VS Code:

```text
Explorer       → files and directories
Editor         → .py, .md, .ipynb, .qmd files
Terminal       → shell commands
Source Control → Git changes
Ports          → forwarded web servers such as Quarto preview
```

### Question

Why is it useful that these tools all operate on the same repository and computational environment?

---

# Part 3 — Explore the repository structure

From the repository root, run:

```bash
ls
```

You may already have directories such as:

```text
data/
notebooks/
src/
figures/
tests/
reports/
```

Create any of these that are missing:

```bash
mkdir -p data notebooks src figures tests reports
```

### Important

All course paths are written relative to the **repository root**.

For example:

```text
data/example.csv
```

is preferred to a computer-specific absolute path.

---

# Part 4 — Create your first Python program

Inside `src/`, create:

```text
hello_sta556.py
```

Add:

```python
print("Hello from STA 556!")

name = "Your Name"

print(f"Welcome to the course, {name}.")
```

From the repository root, run:

```bash
python src/hello_sta556.py
```

### Challenge

Modify the program so that it also prints:

- the course number;
- the semester;
- your Python version.

Hint:

```python
import sys

print(sys.version)
```

---

# Part 5 — Create your first notebook

Inside `notebooks/`, create:

```text
week01_exploration.ipynb
```

If VS Code asks you to choose a kernel, select the Python environment supplied by the Codespace.

Create a Markdown cell:

```markdown
# STA 556 Week 1 Exploration

This notebook is part of my Week 1 computational workflow.
```

Create a code cell:

```python
import sys

print(sys.version)
```

Then:

```python
x = 10
y = 25

x + y
```

Then:

```python
import math

math.sqrt(144)
```

### Reflection

What is different about running:

```text
src/hello_sta556.py
```

and working in:

```text
notebooks/week01_exploration.ipynb
```

Write 2–3 sentences in a Markdown cell.

---

# Part 6 — Add a small data-science example

In the notebook:

```python
scores = [
    72,
    81,
    91,
    68,
    88,
    95,
    77,
    84
]
```

Calculate:

```python
sum(scores) / len(scores)
```

Then:

```python
import numpy as np

scores = np.array(
    scores
)

scores.mean()
scores.std()
```

Create a plot:

```python
import matplotlib.pyplot as plt

plt.hist(
    scores
)
plt.xlabel(
    "Score"
)
plt.ylabel(
    "Frequency"
)
plt.title(
    "Example Score Distribution"
)
plt.show()
```

Save it using a relative path:

```python
plt.hist(
    scores
)
plt.savefig(
    "figures/scores.png",
    dpi=300,
    bbox_inches="tight"
)
plt.show()
```

---

# Part 7 — Understand Git state

Run:

```bash
git status
```

Git should show the files you created or modified.

The repository already has a Git history. Inspect it:

```bash
git log --oneline -5
```

### Question

What is the difference between:

```text
a file existing in the Codespace
```

and:

```text
a file being committed to Git
```

---

# Part 8 — Check `.gitignore`

Look for:

```text
.gitignore
```

A typical course `.gitignore` should exclude items such as:

```text
__pycache__/
.ipynb_checkpoints/
.env
.DS_Store
.pytest_cache/
```

### Important idea

`.gitignore` tells Git not to track specified files. It does not delete them.

---

# Part 9 — Make your first course commit

Run:

```bash
git status
```

Stage your Week 1 work:

```bash
git add .
```

Check again:

```bash
git status
```

Commit:

```bash
git commit -m "Complete Week 1 workflow activity"
```

Inspect the history:

```bash
git log --oneline -5
```

---

# Part 10 — Push to GitHub

Push your commit:

```bash
git push
```

Refresh the GitHub repository in your browser.

Verify that your new files and commit appear there.

### Why this is simpler in Codespaces

You do not need to manually create a Git remote for the repository you opened as a Codespace. The repository was already cloned from GitHub and normally has its remote configured.

Check:

```bash
git remote -v
```

---

# Part 11 — Make a meaningful second change

Open:

```text
src/hello_sta556.py
```

Add:

```python
print(
    "I am beginning to build reproducible computational workflows."
)
```

Save it.

Run:

```bash
git status
git diff
```

Then:

```bash
git add src/hello_sta556.py
git commit -m "Add workflow message to Week 1 script"
git push
```

### Question

What information did `git diff` provide before the commit?

---

# Part 12 — Write useful repository documentation

Open the repository `README.md` (or create one if your assignment repository does not already contain one).

Add a short section describing your Week 1 work, for example:

```markdown
## Week 1 workflow

This project uses the STA 556 GitHub Codespaces environment.

Week 1 demonstrates:

- repository-relative paths
- Python scripts
- Jupyter notebooks
- Git version control
- GitHub synchronization
```

Commit and push the change.

---

# Part 13 — Reproducibility check

Imagine another student opens the same repository in a fresh Codespace.

They should be able to:

```bash
python src/hello_sta556.py
```

and open:

```text
notebooks/week01_exploration.ipynb
```

without knowing anything about your personal laptop.

### Reflection

Why is this a stronger reproducibility test than saying:

> "It runs on my computer"?

---

# Part 14 — Understand persistence

Your repository files live under:

```text
/workspaces/
```

Changes there persist when a Codespace is stopped and restarted.

But a file that has never been committed/pushed is not part of the GitHub repository history.

Explain the difference among:

```text
saved in Codespace
committed to Git
pushed to GitHub
```

---

# Part 15 — Stop and reopen your Codespace

At the end of your work session:

1. make sure important work is committed and pushed;
2. stop the Codespace using GitHub's Codespaces controls rather than simply relying on closing the browser tab.

Later, reopen the same Codespace and run:

```bash
pwd
git status
```

Confirm your work is still present.

---

# Part 16 — Challenge exercises

Complete at least two.

## Challenge A — Project metadata

Create:

```text
src/project_info.py
```

with:

```python
course = "STA 556"
semester = "Fall 2026"
language = "Python"
```

Write a function that prints a useful description.

## Challenge B — Command-line notes

Create:

```text
notebooks/command_line_notes.ipynb
```

Document five shell commands and explain what each does.

## Challenge C — Explore Git history

Run:

```bash
git log --oneline
```

Then investigate one commit with:

```bash
git show <commit-id>
```

## Challenge D — Deliberately break something

Introduce an error in your Python script, run it, read the traceback, fix the error, then commit the correction.

## Challenge E — Portability check

Search your code for any path beginning with something like:

```text
C:\\Users\\...
/Users/...
```

Replace it with a repository-relative path.

---

# Part 17 — Final reflection

Answer in your README or notebook.

### 1. Codespaces

What is a GitHub Codespace, and why does STA 556 use one?

### 2. Git vs. GitHub

Explain the difference between Git and GitHub.

### 3. Repository root

Why do course instructions assume commands are run from the repository root?

### 4. Commit vs. push

What is the difference between committing and pushing?

### 5. Notebook vs. script

When would you choose a Jupyter notebook? When would you choose a Python script?

### 6. Reproducibility

Why are relative paths and a repository-defined environment useful?

---

# Completion checklist

- [ ] Opened the course repository in GitHub Codespaces
- [ ] Located the repository under `/workspaces`
- [ ] Used the integrated terminal
- [ ] Verified Python and Git
- [ ] Explored the VS Code interface
- [ ] Created/run a Python script
- [ ] Created/run a Jupyter notebook
- [ ] Used repository-relative paths
- [ ] Inspected `git status`
- [ ] Inspected Git history
- [ ] Reviewed `.gitignore`
- [ ] Made a meaningful commit
- [ ] Pushed to GitHub
- [ ] Used `git diff`
- [ ] Updated repository documentation
- [ ] Completed a reproducibility check
- [ ] Explained saved vs. committed vs. pushed
- [ ] Stopped/reopened the Codespace

---

# What you should now understand

```text
GitHub repository
      ↓
GitHub Codespace
      ↓
VS Code + Linux shell + Python
      ↓
notebooks / src / data / tests
      ↓
Git commits
      ↓
git push
      ↓
reproducible project history
```
