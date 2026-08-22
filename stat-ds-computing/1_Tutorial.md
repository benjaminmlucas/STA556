# 1.1 Activity

## Build Your First Reproducible Python Data Science Project

**Tools:** Python, VS Code, Jupyter, Git, GitHub, terminal

## Learning objectives

By the end of this tutorial, you should be able to:

- navigate a project using the command line;
- create a sensible data science project structure;
- create and run a Python script;
- create and run a Jupyter notebook;
- initialize a Git repository;
- make commits with meaningful messages;
- connect a local repository to GitHub;
- push your work to GitHub;
- explain why each component belongs in the workflow.

---

# Part 0 — Check your environment

Open a terminal.

Run:

```bash
python --version
```

If that does not work, try:

```bash
python3 --version
```

Then check Git:

```bash
git --version
```

Finally, check that VS Code is available:

```bash
code --version
```

If `code` is not recognized, you can still use VS Code through the graphical interface.

### Checkpoint

You should know:

- your Python version;
- your Git version;
- where VS Code is installed.

---

# Part 1 — Create a project directory

Choose a location where you keep coursework.

Create a directory for STA 556:

```bash
mkdir STA556
cd STA556
```

Now create a project:

```bash
mkdir week01-workflow
cd week01-workflow
```

Check your location:

```bash
pwd
```

List the contents:

```bash
ls
```

The directory should currently be empty.

---

# Part 2 — Build a project structure

Create the directories:

```bash
mkdir data
mkdir notebooks
mkdir src
mkdir figures
mkdir tests
```

Your project should now look approximately like:

```text
week01-workflow/
├── data/
├── notebooks/
├── src/
├── figures/
└── tests/
```

Create a README:

```bash
touch README.md
```

If you are on Windows and `touch` is unavailable, create the file from VS Code instead.

---

# Part 3 — Open the project in VS Code

From the project directory:

```bash
code .
```

If that command does not work, open VS Code normally and select:

**File → Open Folder → week01-workflow**

The important idea is that VS Code should open the **project directory**, not merely an individual Python file.

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

Run it from the terminal.

From the project root:

```bash
python src/hello_sta556.py
```

You should see something similar to:

```text
Hello from STA 556!
Welcome to the course, Your Name.
```

### Challenge

Modify the program so that it also prints:

- the course number;
- the semester;
- your Python version.

For the Python version, investigate:

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

In the notebook, create a Markdown cell containing:

```markdown
# STA 556 Week 1 Exploration

This notebook is part of my Week 1 computational workflow.
```

Then create a code cell:

```python
import sys

print(sys.version)
```

Run the cell.

Now create another code cell:

```python
x = 10
y = 25

x + y
```

Then create another:

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

In the notebook, create a small dataset:

```python
scores = [72, 81, 91, 68, 88, 95, 77, 84]
```

Calculate:

```python
sum(scores) / len(scores)
```

Then import NumPy:

```python
import numpy as np

scores = np.array(scores)

scores.mean()
```

Calculate the standard deviation:

```python
scores.std()
```

Create a simple plot:

```python
import matplotlib.pyplot as plt

plt.hist(scores)
plt.xlabel("Score")
plt.ylabel("Frequency")
plt.title("Example Score Distribution")
plt.show()
```

### Discussion question

Why might it be useful to keep exploratory analysis like this in a notebook rather than putting everything into a Python script?

---

# Part 7 — Initialize Git

Return to the terminal and make sure you are in the project root.

Check:

```bash
pwd
```

Then:

```bash
git init
```

Git should report that an empty repository has been initialized.

Check its status:

```bash
git status
```

Notice that Git can now see the files in your project.

---

# Part 8 — Create a `.gitignore`

Some files should generally **not** be committed to Git.

Create:

```text
.gitignore
```

For this introductory project, add:

```text
__pycache__/
.ipynb_checkpoints/
.env
.DS_Store
```

Later we will discuss more sophisticated Python-specific `.gitignore` files.

### Important idea

A `.gitignore` file tells Git:

> Do not track these files.

It does **not** delete the files.

---

# Part 9 — Make your first commit

Check the repository:

```bash
git status
```

Add the project files:

```bash
git add .
```

Check again:

```bash
git status
```

Notice that files have moved into the **staging area**.

Now commit:

```bash
git commit -m "Create initial STA 556 project structure"
```

Check the history:

```bash
git log --oneline
```

You should see your commit.

---

# Part 10 — Make a meaningful change

Go back to:

```text
src/hello_sta556.py
```

Add another line:

```python
print("I am beginning to build reproducible computational workflows.")
```

Save the file.

Now run:

```bash
git status
```

Git should report that the file has been modified.

Look at the difference:

```bash
git diff
```

This is an important command.

It lets you ask:

> What exactly changed since my last commit?

Now stage and commit the change:

```bash
git add src/hello_sta556.py
git commit -m "Add workflow message to Python script"
```

View the history:

```bash
git log --oneline
```

You should now have at least two commits.

---

# Part 11 — Connect the project to GitHub

Create a new **empty repository** on GitHub called something like:

```text
sta556-week01-workflow
```

Do not initialize it with another README if you are following this tutorial exactly.

GitHub will provide a repository URL.

Connect your local project:

```bash
git remote add origin YOUR_GITHUB_REPOSITORY_URL
```

Verify:

```bash
git remote -v
```

You should see your GitHub repository listed.

---

# Part 12 — Push your project

Rename your local default branch to `main` if necessary:

```bash
git branch -M main
```

Push:

```bash
git push -u origin main
```

Refresh GitHub in your browser.

You should now see your project.

---

# Part 13 — Explore your GitHub repository

Look at the repository as if you were another researcher encountering it for the first time.

Can you determine:

- what the project is?
- who created it?
- what files it contains?
- how the files are organized?
- what the project does?
- what has changed?

If the answer is "no," your next task is to improve the README.

---

# Part 14 — Write a useful README

Open `README.md`.

Add:

```markdown
# STA 556 Week 1 Workflow

This repository contains my Week 1 work for STA 556:
Statistics and Data Science Computing Workflows.

## Project structure

- `data/` — data files
- `notebooks/` — Jupyter notebooks
- `src/` — Python source code
- `figures/` — generated figures
- `tests/` — tests

## Environment

- Python 3.x
- NumPy
- Matplotlib
- Jupyter

## Contents

The project demonstrates:

- command-line navigation
- Python scripts
- Jupyter notebooks
- Git version control
- GitHub
```

Save it.

Then:

```bash
git status
git diff
```

Commit:

```bash
git add README.md
git commit -m "Document project structure and environment"
```

Push:

```bash
git push
```

---

# Part 15 — A mini reproducibility test

This is the most important exercise.

Imagine that a classmate has just cloned your repository.

Ask yourself:

> Could they understand what this project is and how to begin using it?

If possible, have a partner clone your repository:

```bash
git clone YOUR_GITHUB_REPOSITORY_URL
```

Then:

```bash
cd sta556-week01-workflow
```

Try to run:

```bash
python src/hello_sta556.py
```

Then open the notebook.

### If your partner cannot do this...

Do not immediately fix their computer.

First ask:

> What information is missing from my project?

This is the beginning of thinking like a data scientist who builds **reproducible workflows**.

---

# Part 16 — Challenge exercises

Complete at least two.

## Challenge A — Add a project metadata file

Create:

```text
src/project_info.py
```

Define variables for:

```python
course = "STA 556"
semester = "Fall 2026"
language = "Python"
```

Write a function:

```python
def describe_project(course, semester, language):
    ...
```

that prints a useful description.

Commit the result.

---

## Challenge B — Add another notebook

Create:

```text
notebooks/command_line_notes.ipynb
```

Document five terminal commands and explain what each does.

---

## Challenge C — Explore Git history

Run:

```bash
git log --oneline
```

Then:

```bash
git diff HEAD~1
```

What changed in the most recent commit?

---

## Challenge D — Deliberately break something

Modify your Python script so that it contains an error.

Run it.

Record the error message.

Fix the problem.

Then commit the correction.

The goal is to recognize that error messages are **information**, not failure.

---

## Challenge E — Create a reproducible figure

In the notebook, generate a plot from the `scores` data.

Save the figure to:

```text
figures/scores.png
```

Hint:

```python
plt.savefig("figures/scores.png", dpi=300, bbox_inches="tight")
```

Then commit the figure.

---

# Part 17 — Final reflection

Answer these questions in your README or notebook.

### 1. Git vs. GitHub

In your own words, explain the difference between Git and GitHub.

### 2. Working directory vs. repository

What is the difference between a file that exists in your project folder and a file that has been committed to Git?

### 3. Why commit?

Why is it useful to create many small, meaningful commits rather than one enormous commit at the end of a project?

### 4. Notebook vs. script

When would you choose a Jupyter notebook? When would you choose a Python script?

### 5. Reproducibility

What information would another researcher need to reproduce your analysis?

---

# Completion checklist

- [ ] Python runs from the terminal
- [ ] Git runs from the terminal
- [ ] STA 556 project directory created
- [ ] Project subdirectories created
- [ ] Python script created and executed
- [ ] Jupyter notebook created and executed
- [ ] `.gitignore` created
- [ ] Git repository initialized
- [ ] At least three meaningful commits created
- [ ] GitHub repository created
- [ ] Local repository connected to GitHub
- [ ] Project pushed to GitHub
- [ ] README written
- [ ] Reproducibility test attempted
- [ ] Reflection questions answered

---

# What you should now understand

The central lesson is not a particular Git command.

It is this workflow:

```text
Organize
   ↓
Write
   ↓
Run
   ↓
Inspect
   ↓
Document
   ↓
Commit
   ↓
Share
   ↓
Reproduce
```

This workflow will become the foundation for the rest of STA 556.
