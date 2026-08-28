# 1.1 Activity

## Build Your First Reproducible Python Data Science Project in GitHub Codespaces

**Tools:** GitHub Codespaces, Python, VS Code, Jupyter, Git, GitHub, integrated terminal

## Learning objectives

By the end of this tutorial, you should be able to:

- create a new GitHub repository for your STA 556 course work;
- create and navigate a GitHub Codespace from that repository;
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

# Part 0 — Create your STA 556 GitHub repository

For this course, you will begin by creating a GitHub repository that will hold your STA 556 work.

If you are not already signed in to GitHub, go to:

```text
https://github.com
```

and sign in.

If you do not yet have a GitHub account, choose **Sign up**, create an account, complete any requested email verification, and then sign in.

## 0.1 Create a new repository

From GitHub:

1. click the **+** menu in the upper-right corner;
2. choose **New repository**;
3. give the repository a name such as:

```text
STA556
```

If you already have a repository with that name, you could use:

```text
STA556-Fall-2026
```

4. optionally add a short description such as:

```text
Course work for STA 556: Statistics and Data Science Computing Workflows
```

5. choose **Private** unless your instructor has asked you to make the repository public;
6. check **Add a README file**;
7. for `.gitignore`, choose **Python** if GitHub offers that option;
8. you do not need to add a license for this activity;
9. click **Create repository**.

### Why add a README?

A README gives the repository an initial file and creates the default branch immediately. It also gives you a natural place to document the project as the semester progresses.

---

## 0.2 Create a Codespace from your new repository

On the page for the repository you just created:

1. click the green **Code** button;
2. choose the **Codespaces** tab;
3. click **Create codespace on main**.

GitHub will create a cloud development environment attached to **your new STA 556 repository**.

The first Codespace may take a few minutes to start.

When VS Code opens in the browser, open an integrated terminal:

```text
Terminal → New Terminal
```

Run:

```bash
pwd
```

You should see something similar to:

```text
/workspaces/STA556
```

or:

```text
/workspaces/STA556-Fall-2026
```

The exact repository name may differ, but the path should begin with:

```text
/workspaces/
```

Now run:

```bash
git status
```

Git should recognize the repository.

### Important

When you create a Codespace from a GitHub repository, GitHub automatically places that repository inside the Codespace and configures Git for it.

**Do not run `git init`.**

You already have a Git repository.

---

# Part 1 — Set up and check your Week 1 Python environment

Because you created a new repository, you will also record the small set of Python packages needed for this activity.

From the repository root, create:

```text
requirements.txt
```

Add:

```text
numpy
pandas
matplotlib
jupyter
ipykernel
```

Save the file.

Now install the packages listed in it:

```bash
python -m pip install -r requirements.txt
```

This may take a little time the first time.

Now check Python and Git:

```bash
python --version
git --version
```

Then verify the main packages:

```bash
python -c "import numpy, pandas, matplotlib; print('Core packages available')"
```

You should see:

```text
Core packages available
```

### Why create `requirements.txt`?

Rather than remembering package names or installing them randomly, you have written the project's dependencies into a file that is stored with the repository.

Another person can later see that the project depends on:

```text
numpy
pandas
matplotlib
jupyter
ipykernel
```

and install them with the same command.

Later in the course, we will build on this idea and discuss more complete approaches to reproducible computational environments.

### Checkpoint

At this point you should know:

- the name of your STA 556 GitHub repository;
- how you created a Codespace from that repository;
- your current working directory;
- your Python version;
- your Git version;
- that your repository is already under version control;
- where your project dependencies are recorded;
- that the packages needed for this activity are available.

---

# Part 2 — Explore the Codespaces interface

Identify the following parts of VS Code:

```text
Explorer       → files and directories
Editor         → .py, .md, and .ipynb files
Terminal       → shell commands
Source Control → Git changes
```

### Question

Why is it useful that these tools all operate on the same repository and computational environment?

---

# Part 3 — Explore the repository structure

From the repository root, run:

```bash
ls
```

Your new repository will initially contain only a few files.

Create the following project directories:

```bash
mkdir -p data notebooks src figures
```

Then run:

```bash
ls
```

You should now have a simple course-project structure similar to:

```text
README.md
requirements.txt
data/
notebooks/
src/
figures/
```

We will add other project directories later in the semester when we need them.

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

Your repository already has a small Git history beginning with its creation. Inspect it:

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

Open the `README.md` that you created when you created the repository.

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

Imagine you delete this Codespace and create a completely fresh Codespace from the same GitHub repository.

The repository contains `requirements.txt`, so you could first restore the packages with:

```bash
python -m pip install -r requirements.txt
```

Then you should be able to:

```bash
python src/hello_sta556.py
```

and open:

```text
notebooks/week01_exploration.ipynb
```

without relying on anything installed on your personal laptop.

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

- [ ] Created a new STA 556 GitHub repository
- [ ] Created a Codespace from that repository
- [ ] Located the repository under `/workspaces`
- [ ] Created and used `requirements.txt`
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
Create your STA 556 GitHub repository
      ↓
Create a Codespace from that repository
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
