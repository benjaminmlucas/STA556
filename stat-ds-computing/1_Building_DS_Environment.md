# 1.1 Building a Professional Python Data Science Environment

## Why this week matters

STA 556 is not primarily a course about learning Python syntax. It is a course about developing **professional computational workflows for statistics and data science**.

The course syllabus identifies Week 1 as the foundations module, covering:

- Command-line basics (Bash/Zsh)
- Git and GitHub for version control
- IDE setup
- Reproducible computational work

These skills support two important course outcomes:

1. Configure and manage professional development environments using version control and IDEs.
2. Execute basic system operations via the command line to automate file management and script execution.

The broader goal is to move from:

> "I can write some Python code."

to:

> "I can organize, run, document, version, reproduce, and share a statistical computing project."

That distinction will become increasingly important as the course moves into data wrangling, simulation, optimization, testing, and computational efficiency.

---

# 1. The computational workflow

A modern data science project is rarely a single notebook.

A more realistic workflow looks like:

```text
Research question
      ↓
Project directory
      ↓
Data ──→ Python code ──→ Results
             ↓              ↓
          Git history     Figures/tables
             ↓              ↓
          GitHub       Report/notebook
```

A professional workflow separates different kinds of material:

```text
my-project/
├── README.md
├── .gitignore
├── data/
├── notebooks/
├── src/
├── tests/
├── figures/
└── environment/
```

We will not use every directory immediately, but students should begin thinking of a data science project as a **structured computational artifact**, rather than a collection of files sitting on a desktop.

## Why structure matters

A structured project makes it easier to:

- find code and data;
- reproduce an analysis;
- collaborate with another researcher;
- identify what changed;
- move a project to another computer;
- debug errors;
- automate repeated tasks;
- eventually publish computational work.

The course's emphasis on reproducible research, Git/GitHub, Jupyter, and robust code is built around this idea.

---

# 2. The command line: talking directly to your computer

A graphical interface lets us interact with a computer by clicking.

A command-line interface (CLI) lets us interact with the computer by **typing commands**.

For example:

```bash
pwd
```

asks:

> Where am I?

```bash
ls
```

asks:

> What is here?

```bash
cd my-project
```

means:

> Move into the `my-project` directory.

```bash
mkdir data
```

means:

> Create a directory called `data`.

The important conceptual shift is that the command line gives us a **language for manipulating the filesystem**.

## Why should data scientists learn the command line?

The command line becomes especially useful when:

- working with large numbers of files;
- running scripts;
- automating repetitive operations;
- connecting to remote computers;
- working on high-performance computing systems;
- using Git;
- building reproducible pipelines.

Software Carpentry makes this point particularly well: the shell is valuable not because it replaces graphical tools, but because it allows us to combine small operations and automate repetitive work.

### Essential commands

| Command | Purpose |
|---|---|
| `pwd` | Print working directory |
| `ls` | List files/directories |
| `cd` | Change directory |
| `mkdir` | Make a directory |
| `touch` | Create an empty file |
| `cp` | Copy |
| `mv` | Move/rename |
| `rm` | Remove |
| `cat` | Display file contents |
| `clear` | Clear terminal |
| `history` | Show previous commands |

### Absolute vs. relative paths

An **absolute path** specifies a location from the root of the filesystem.

A **relative path** specifies a location relative to your current working directory.

For example:

```text
/Users/ben/projects/sta556/data
```

is absolute.

Whereas:

```text
data/
```

might be a relative path from the project directory.

Understanding paths is fundamental to reproducible data science because code should not depend on a particular person's computer.

---

# 3. Git is not GitHub

These terms are often confused.

## Git

**Git is a version control system.**

It records changes to files over time.

Imagine writing a research paper and saving:

```text
analysis_final.py
analysis_final2.py
analysis_final_really_final.py
analysis_final_really_final_USE_THIS.py
```

Git provides a much better solution.

It lets us record meaningful versions of the project:

```text
commit 1 → commit 2 → commit 3 → commit 4
```

Each commit represents a snapshot of the project.

## GitHub

**GitHub is a platform for hosting Git repositories and collaborating around them.**

A useful mental model is:

```text
Git
│
├── version control on your computer
│
└── GitHub
    └── remote hosting + collaboration
```

You can use Git without GitHub.

You can also use GitHub as much more than file storage: repositories can contain documentation, issues, pull requests, discussions, and project history.

---

# 4. The basic Git workflow

The basic workflow is:

```text
Working directory
       ↓
   git add
       ↓
Staging area
       ↓
 git commit
       ↓
Repository
       ↓
  git push
       ↓
   GitHub
```

The commands we will use repeatedly are:

```bash
git status
git add .
git commit -m "Meaningful description of change"
git push
```

## `git status`

This is one of the most useful Git commands.

```bash
git status
```

asks Git:

> What has changed?

Use it frequently.

## `git add`

```bash
git add my_file.py
```

tells Git:

> Include this change in my next commit.

## `git commit`

```bash
git commit -m "Add data loading function"
```

records a snapshot of the staged changes.

A good commit message explains **what changed**.

Bad:

```text
stuff
```

Better:

```text
Add function to load census data
```

## `git push`

```bash
git push
```

sends your local commits to the remote repository on GitHub.

---

# 5. Why version control matters in statistics

Version control is not only a software engineering practice.

It is also a **research reproducibility tool**.

Suppose you obtain a result:

```text
Estimated effect = 0.37
```

Three months later, the result is:

```text
Estimated effect = 0.42
```

What changed?

Without version control, answering that question may be difficult.

With Git, you can inspect the history of:

- analysis code;
- data-processing code;
- configuration files;
- documentation;
- statistical methods;
- visualizations.

This creates an audit trail for computational decisions.

Git does not automatically make an analysis reproducible. It is one component of a reproducible workflow.

---

# 6. The Python development environment

For this semester, we will use **Python** as the programming language for STA 556.

The main development environment will be **Visual Studio Code (VS Code)** and/or **Jupyter**, depending on the task.

## VS Code

VS Code is a general-purpose code editor/IDE.

It is particularly useful for:

- `.py` files;
- project organization;
- Git integration;
- debugging;
- testing;
- working across multiple files;
- managing larger projects.

## Jupyter

Jupyter notebooks are useful for:

- exploratory analysis;
- interactive computation;
- visualization;
- explaining computational reasoning;
- combining code, output, and narrative.

A notebook is therefore particularly useful when we want to tell the story of an analysis.

However, notebooks can also become difficult to maintain if they are treated as the entire software project.

A useful distinction is:

> **Notebook = interactive analysis environment**

> **Python module/script = reusable computational code**

We will use both.

---

# 7. Environments and dependencies

A Python program often depends on external packages.

For example:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

Another computer may not have exactly the same packages or versions installed.

This creates a reproducibility problem.

A professional workflow therefore records the computational environment.

For example:

```text
Python 3.x
numpy
pandas
matplotlib
scipy
scikit-learn
```

Later in the course we will develop more systematic approaches to managing dependencies and environments.

The key principle for Week 1 is:

> **Your code is only one part of a computational analysis. The environment in which the code runs also matters.**

---

# 8. What makes a good project?

A good computational project should answer four questions.

### 1. What is this?

The `README.md` should provide a concise description.

### 2. How do I run it?

Another person should be able to determine how to reproduce the analysis.

### 3. What changed?

Git provides the history.

### 4. Where are the important components?

A predictable project structure makes the answer obvious.

---

# 9. Recommended reading

## Primary course references

The syllabus identifies the following as primary references:

### Jake VanderPlas — *Python Data Science Handbook*

A strong reference for the Python data science ecosystem, particularly NumPy, Pandas, Matplotlib, and scikit-learn.

https://jakevdp.github.io/PythonDataScienceHandbook/

For Week 1, focus on the introductory material and the general philosophy of the Python data science stack rather than attempting to read the entire book.

### Wes McKinney — *Python for Data Analysis*, 3rd ed.

A particularly useful reference for the data-analysis workflow that we will develop later in the course.

https://wesmckinney.com/book/

---

## Additional recommended reading

### Software Carpentries — The Unix Shell

This is an excellent practical introduction to the command line. It begins from the assumption that students may have little or no prior shell experience.

https://swcarpentry.github.io/shell-novice/

Recommended sections:

- Introducing the Shell
- Navigating Files and Directories
- Working With Files and Directories
- Pipes and Filters

### Software Carpentries — Version Control with Git

A complementary introduction to Git that emphasizes the research workflow.

https://swcarpentry.github.io/git-novice/

### GitHub Skills

GitHub provides interactive exercises for learning GitHub workflows.

https://skills.github.com/

---

# 10. YouTube recommendations

## 1. GitHub — "A brief introduction to Git for beginners"

This is my first recommendation because it comes directly from GitHub and provides a concise conceptual introduction to Git, including version control, repositories, staging, committing, branches, and the distinction between Git and GitHub.

urlWatch on YouTubehttps://www.youtube.com/watch?v=r8jQ9hVA2qs

**Use this before the activity.**

## 2. Software Carpentry — "The Shell - Episode 1 - Introduction"

A short introduction to what a shell is and why computational researchers should care about it.

urlWatch on YouTubehttps://www.youtube.com/watch?v=U3iNcBtycaQ

**Use this before the command-line portion of the activity.**

## 3. Kevin Stratvert — "Git and GitHub Tutorial for Beginners"

A longer, very practical walkthrough. It covers initializing repositories, staging, committing, branches, GitHub, pushing/pulling, and typical Git workflows.

urlWatch on YouTubehttps://www.youtube.com/watch?v=tRZGeaHPoaw

**Use this as an optional extension/reference rather than required viewing.**

---

# Week 1 takeaway

By the end of Week 1, students should be able to explain:

1. What a command-line shell is and why it is useful.
2. The difference between Git and GitHub.
3. The difference between a working directory, staging area, and repository.
4. Why version control is valuable for statistical research.
5. The roles of VS Code and Jupyter.
6. Why computational environments and dependencies matter.
7. What a professional data science project structure looks like.

Most importantly, students should leave Week 1 thinking about **data science as a computational workflow**, rather than simply as writing code.
