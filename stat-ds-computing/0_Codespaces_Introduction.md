# GitHub Codespaces for STA 556

## Why STA 556 uses Codespaces

STA 556 students may be working from Macs, Windows PCs, or university computers on which they do not have administrator rights. To give everyone a common computational environment, the officially supported environment for the course is **GitHub Codespaces**.

A Codespace runs the development environment remotely and gives you VS Code in a web browser.

```text
Your computer
   ↓
Web browser
   ↓
GitHub Codespaces
   ↓
Linux + Python + Jupyter + Git + course packages
   ↓
Your GitHub repository
```

This means the machine in front of you mainly acts as an interface. The course code runs in the common Codespace environment.

---

# 1. Before you begin

You need:

- a GitHub account;
- access to the STA 556 repository supplied by your instructor;
- a modern web browser;
- an internet connection.

You do **not** need administrator access on the computer you are using.

---

# 2. Create a Codespace

Open the repository on GitHub.

Use the repository's **Code** menu, choose **Codespaces**, and create a Codespace for the repository. If you already created one, reopen it rather than creating a new environment each time.

Creation may take a little longer the first time because the course container and Python dependencies are configured automatically.

When ready, VS Code opens in the browser.

---

# 3. The VS Code interface

The main areas you will use are:

```text
Explorer
    files and directories

Editor
    Python, Markdown, notebooks, Quarto

Terminal
    shell, Python, Git, pytest, Quarto

Source Control
    Git changes and commits

Ports
    browser access to preview servers
```

You do not need to memorize the entire VS Code interface. The course will introduce features as they are needed.

---

# 4. Find the repository root

Open an integrated terminal and run:

```bash
pwd
```

You should see something similar to:

```text
/workspaces/sta556
```

The exact repository name may differ.

Run:

```bash
ls
```

and:

```bash
git status
```

`git status` should recognize the repository.

The **repository root** is the main working directory for STA 556. Unless the course says otherwise, run commands from here.

---

# 5. Relative paths

Use repository-relative paths in code.

Good:

```python
import pandas as pd

survey = pd.read_csv(
    "data/raw/survey.csv"
)
```

Avoid paths tied to one personal computer:

```text
C:\\Users\\Name\\Desktop\\survey.csv
/Users/name/Desktop/survey.csv
```

Relative paths make your project portable.

---

# 6. Run Python

Check Python:

```bash
python --version
```

Run a script from the repository root:

```bash
python src/example.py
```

In STA 556 documentation, `python` is the standard command used inside the Codespace.

---

# 7. Work with Jupyter notebooks

You can open `.ipynb` files directly in VS Code.

When asked to choose a kernel, select the Python environment supplied by the Codespace.

You normally do **not** need to start a separate `jupyter notebook` or `jupyter lab` server.

A useful reproducibility check is:

```text
Restart kernel
      ↓
Run all
      ↓
Notebook succeeds from top to bottom
```

---

# 8. Work from the repository root

Some later course material imports reusable modules:

```python
from src.analysis import summarize
```

Keep the repository root open as the VS Code workspace. Do not `cd notebooks` and start an unrelated Jupyter server there.

This course convention keeps relative paths and imports predictable.

---

# 9. Git workflow

The repository is already a Git repository when it opens in Codespaces.

Do **not** run:

```bash
git init
```

inside it.

Your routine workflow is:

```bash
git status
git add .
git commit -m "Meaningful description"
git push
```

Use:

```bash
git diff
```

before committing when you want to inspect exactly what changed.

Use:

```bash
git log --oneline -5
```

to inspect recent history.

---

# 10. Saved vs. committed vs. pushed

These are different states.

## Saved

The file exists in your Codespace filesystem.

## Committed

Git has recorded the change in the repository history.

## Pushed

The commit has been sent back to GitHub.

A useful habit is:

```text
save frequently
commit meaningful units of work
push regularly
```

---

# 11. What persists?

The repository is stored under `/workspaces`, which is the persistent workspace area for your Codespace.

Files there persist when you stop and restart the Codespace and across container rebuilds.

However:

> **Persistence is not the same as version control.**

If important work has not been committed and pushed, it is not part of the GitHub repository history.

---

# 12. Course software and dependencies

The repository defines the course environment using:

```text
.devcontainer/devcontainer.json
requirements.txt
```

When the Codespace is created, the required tools are installed automatically.

Therefore, if an import such as:

```python
import scipy
```

fails in the official course Codespace, do not immediately run an arbitrary installation command. First:

1. check that you opened the correct repository/Codespace;
2. check the course instructions;
3. report the environment problem if it persists.

This helps keep the class on a common environment.

---

# 13. Quarto previews and forwarded ports

Later in the course you may run:

```bash
quarto preview reports/report.qmd
```

The preview server is running **inside the Codespace**.

Codespaces detects server ports and provides a forwarded browser-accessible URL. Use the **Open in Browser** notification or the **Ports** panel in VS Code.

Do not assume that a `localhost` URL printed in the terminal should be typed directly into your laptop browser.

---

# 14. Secrets and API keys

Never commit an API key, password, or token to Git.

Do not write:

```python
api_key = "my-secret-key"
```

in a notebook that will be committed.

For authenticated services, the course will use environment variables or Codespaces secrets when needed.

Python can read an environment variable with:

```python
import os

api_key = os.environ[
    "MY_API_KEY"
]
```

A `.env` file, if used locally for temporary configuration, should remain ignored by Git.

---

# 15. Stop your Codespace

When you finish working:

1. save your files;
2. commit and push important work;
3. stop the Codespace using GitHub's Codespaces controls.

Closing the browser tab is not the same as deliberately stopping the remote environment.

You can reopen the same Codespace later.

---

# 16. Rebuild vs. restart

## Restart/reopen

Use this for an ordinary new work session.

## Rebuild container

A rebuild recreates the development container from the repository's `.devcontainer` configuration.

Rebuilding can be useful after the instructor changes course dependencies or container settings.

Files inside `/workspaces` persist across a rebuild, but software installed manually elsewhere in the container may not.

This is another reason not to rely on ad-hoc manual environment changes.

---

# 17. Common troubleshooting

## `git status` says this is not a repository

Check:

```bash
pwd
```

You may have changed into the wrong directory. Return to the repository root.

## `ModuleNotFoundError`

First confirm:

```bash
pwd
```

and that you are using the course Python kernel/environment.

Do not immediately install a different version of the package.

## `from src...` fails in a notebook

Confirm that the repository root is the open VS Code workspace and that you have not launched Jupyter from inside `notebooks/`.

## Notebook uses the wrong kernel

Use the kernel selector in the notebook toolbar and choose the course Python environment.

## Quarto preview opens no page

Look at the **Ports** panel and use the forwarded URL.

## My changes are not on GitHub

Check:

```bash
git status
git log --oneline -5
```

You may have saved the file but not committed/pushed it.

---

# 18. Optional local development

You are welcome to use a local Python/VS Code setup on your own computer if you prefer.

However:

> **The Codespace is the officially supported STA 556 environment.**

Your submitted project should therefore work in the course Codespace even if you developed it locally.

This is a useful portability test.

---

# 19. Quick reference

```bash
# Where am I?
pwd

# What files are here?
ls

# What has changed?
git status

# Show unstaged changes
git diff

# Stage changes
git add .

# Commit
git commit -m "Describe the change"

# Push to GitHub
git push

# Recent history
git log --oneline -5

# Python version
python --version

# Run a script
python src/example.py

# Run tests
pytest

# Render Quarto
quarto render reports/report.qmd
```

---

# Codespaces takeaway

```text
One repository
      ↓
One defined environment
      ↓
Same Linux/Python workflow
      ↓
Mac / PC / university computer all work
      ↓
portable, reproducible course projects
```
