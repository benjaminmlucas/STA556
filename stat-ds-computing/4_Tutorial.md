# 2.3 Activity

## Input/Output & External Data Sources

**Estimated time:** 2–3 hours  
**Tools:** Python, pandas, requests, sqlite3, Jupyter/VS Code

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

- read and write CSV files;
- inspect data after ingestion;
- work with project-relative paths;
- create and query an SQLite database;
- load SQL results into pandas;
- retrieve data from an HTTP API;
- inspect status codes;
- parse and normalize JSON;
- read HTML tables;
- save and load Parquet;
- understand the role of HDF5;
- validate externally sourced data;
- preserve raw data and record provenance.

---

# Part 0 — Create a Week 5 project structure

Create:

```text
data/
├── raw/
├── processed/
└── external/
```

Create:

```text
notebooks/week05_external_data.ipynb
```

Import:

```python
from pathlib import Path
import sqlite3
import json

import pandas as pd
import requests
```

Define:

```python
DATA_DIR = Path("data")
RAW_DIR = DATA_DIR / "raw"
PROCESSED_DIR = DATA_DIR / "processed"
EXTERNAL_DIR = DATA_DIR / "external"
```

---

# Part 1 — Create a source dataset

```python
participants = pd.DataFrame({
    "id": ["001", "002", "003", "004", "005"],
    "age": [24, 31, 27, 42, 35],
    "group": ["A", "B", "A", "B", "A"],
    "score": [81, 94, 88, 91, 85]
})
```

Inspect:

```python
participants
participants.dtypes
```

Why might storing `id` as a string matter?

---

# Part 2 — Write a CSV

```python
csv_path = RAW_DIR / "participants.csv"

participants.to_csv(
    csv_path,
    index=False
)
```

Check:

```python
csv_path.exists()
```

Open the file as text.

### Question

What pandas data-type information is explicitly stored in the CSV?

---

# Part 3 — Read the CSV

```python
loaded = pd.read_csv(csv_path)
```

Inspect:

```python
loaded
loaded.dtypes
```

What happened to:

```text
001
002
003
```

?

This illustrates that file ingestion can alter representation.

---

# Part 4 — Control data types

```python
loaded = pd.read_csv(
    csv_path,
    dtype={
        "id": "string",
        "group": "category"
    }
)
```

Inspect:

```python
loaded.dtypes
loaded
```

Why is an identifier often better represented as a string than as a numeric measurement?

---

# Part 5 — Missing-value codes

```python
messy = pd.DataFrame({
    "id": ["001", "002", "003", "004"],
    "score": ["81", "-999", "88", "NA"]
})
```

Save:

```python
messy_path = RAW_DIR / "messy_scores.csv"

messy.to_csv(
    messy_path,
    index=False
)
```

Read:

```python
messy_loaded = pd.read_csv(
    messy_path,
    na_values=["NA", "-999"]
)
```

Inspect:

```python
messy_loaded
messy_loaded.dtypes
messy_loaded.isna().sum()
```

Why is documenting missing-value coding part of provenance?

---

# Part 6 — Validate after reading

```python
loaded.shape
loaded.head()
loaded.dtypes
loaded.isna().sum()
loaded["id"].nunique()
loaded["group"].value_counts()
```

Add checks:

```python
assert len(loaded) == 5
assert loaded["id"].is_unique
```

```python
assert set(
    loaded["group"].astype(str)
) <= {"A", "B"}
```

> **A successful read operation is only the beginning of ingestion.**

---

# Part 7 — Create an SQLite database

```python
db_path = RAW_DIR / "study.db"

connection = sqlite3.connect(db_path)
```

Write:

```python
participants.to_sql(
    "participants",
    connection,
    if_exists="replace",
    index=False
)
```

---

# Part 8 — Read an SQL table

```python
sql_df = pd.read_sql(
    "SELECT * FROM participants",
    connection
)
```

Inspect:

```python
sql_df
```

What role did SQL play, and what role did pandas play?

---

# Part 9 — Select columns with SQL

```python
query = """
SELECT
    id,
    age,
    score
FROM participants
"""

selected = pd.read_sql(
    query,
    connection
)
```

Compare with:

```python
participants[
    ["id", "age", "score"]
]
```

---

# Part 10 — Filter with SQL

```python
query = """
SELECT *
FROM participants
WHERE age >= 30
"""

older = pd.read_sql(
    query,
    connection
)
```

Compare with:

```python
participants.loc[
    participants["age"] >= 30
]
```

Why might filtering in SQL be advantageous for a very large database?

---

# Part 11 — SQL aggregation

```python
query = """
SELECT
    group,
    COUNT(*) AS n,
    AVG(score) AS mean_score
FROM participants
GROUP BY group
"""

summary = pd.read_sql(
    query,
    connection
)
```

### Challenge

Write SQL returning:

- group;
- minimum score;
- maximum score.

---

# Part 12 — Parameterized SQL

```python
query = """
SELECT *
FROM participants
WHERE group = ?
"""
```

```python
group_a = pd.read_sql(
    query,
    connection,
    params=("A",)
)
```

Why might parameters be preferable to inserting values directly into a query string?

---

# Part 13 — Create another table

```python
outcomes = pd.DataFrame({
    "id": ["001", "002", "003", "005"],
    "followup_score": [85, 97, 91, 89]
})
```

```python
outcomes.to_sql(
    "outcomes",
    connection,
    if_exists="replace",
    index=False
)
```

---

# Part 14 — SQL join

```python
query = """
SELECT
    p.id,
    p.age,
    p.group,
    p.score,
    o.followup_score
FROM participants AS p
LEFT JOIN outcomes AS o
    ON p.id = o.id
"""

joined = pd.read_sql(
    query,
    connection
)
```

Inspect:

```python
joined
joined.isna().sum()
```

Which Week 4 pandas merge is equivalent?

---

# Part 15 — Close the connection

```python
connection.close()
```

Why close resources when they are no longer needed?

---

# Part 16 — Make an HTTP request

Use a public practice API:

```python
url = "https://jsonplaceholder.typicode.com/users"

response = requests.get(
    url,
    timeout=10
)
```

Inspect:

```python
response.status_code
```

Then:

```python
response.raise_for_status()
```

Why should the program check the response before analysis?

---

# Part 17 — Inspect and parse JSON

```python
response.text[:500]
```

Then:

```python
data = response.json()
```

Inspect:

```python
type(data)
len(data)
type(data[0])
data[0]
```

What Python structures does the JSON become?

---

# Part 18 — Convert JSON to a DataFrame

```python
users = pd.DataFrame(data)
```

Inspect:

```python
users.head()
users.columns
```

Look at:

```python
users.loc[0, "address"]
```

Why is this column not yet a simple scalar variable?

---

# Part 19 — Normalize nested JSON

```python
users_flat = pd.json_normalize(data)
```

Inspect:

```python
users_flat.head()
users_flat.columns
```

### Challenge

Select:

```text
id
name
email
address.city
company.name
```

---

# Part 20 — Save the raw API response

```python
json_path = RAW_DIR / "users.json"

with open(
    json_path,
    "w",
    encoding="utf-8"
) as f:
    json.dump(
        data,
        f,
        indent=2
    )
```

Why does saving the response improve reproducibility?

---

# Part 21 — Read the local JSON snapshot

```python
with open(
    json_path,
    "r",
    encoding="utf-8"
) as f:
    local_data = json.load(f)
```

Compare:

```python
local_data == data
```

Now your analysis can use the saved snapshot rather than repeatedly calling the API.

---

# Part 22 — API error experiment

```python
bad_url = (
    "https://jsonplaceholder.typicode.com/"
    "does-not-exist"
)

bad_response = requests.get(
    bad_url,
    timeout=10
)
```

Inspect:

```python
bad_response.status_code
```

Then:

```python
bad_response.raise_for_status()
```

Why is failure handling part of data engineering?

---

# Part 23 — Read HTML tables

Use a page containing tabular data:

```python
url = (
    "https://en.wikipedia.org/wiki/"
    "List_of_states_and_territories_"
    "of_the_United_States"
)

tables = pd.read_html(url)
```

Inspect:

```python
len(tables)
tables[0].head()
```

`read_html()` returns a list because a page may contain multiple tables.

---

# Part 24 — Identify the appropriate table

```python
for i, table in enumerate(tables[:5]):
    print(
        i,
        table.shape,
        list(table.columns)[:4]
    )
```

Why is assuming `tables[0]` unsafe in a reusable workflow?

---

# Part 25 — Save processed data

```python
processed_csv = (
    PROCESSED_DIR / "users_processed.csv"
)

users_flat.to_csv(
    processed_csv,
    index=False
)
```

This separates raw external input from processed analytical data.

---

# Part 26 — Write Parquet

```python
parquet_path = (
    PROCESSED_DIR / "users_processed.parquet"
)

users_flat.to_parquet(
    parquet_path,
    index=False
)
```

Read:

```python
users_parquet = pd.read_parquet(
    parquet_path
)
```

Inspect:

```python
users_parquet.head()
users_parquet.dtypes
```

If a Parquet engine is missing, document the dependency indicated by the error.

---

# Part 27 — Compare file sizes

```python
processed_csv.stat().st_size
parquet_path.stat().st_size
```

Then:

```python
ratio = (
    parquet_path.stat().st_size
    / processed_csv.stat().st_size
)

ratio
```

Should you generalize performance conclusions from such a tiny dataset?

---

# Part 28 — Compare types

```python
csv_again = pd.read_csv(
    processed_csv
)
```

Compare:

```python
csv_again.dtypes
users_parquet.dtypes
```

What advantages can a typed binary format provide?

---

# Part 29 — Optional HDF5 exercise

If PyTables is installed:

```python
hdf_path = (
    PROCESSED_DIR / "users.h5"
)

users_flat.to_hdf(
    hdf_path,
    key="users",
    mode="w"
)
```

Read:

```python
users_hdf = pd.read_hdf(
    hdf_path,
    key="users"
)
```

How does specifying a `key` differ from reading a simple CSV?

If the required dependency is not installed, document that rather than installing software without understanding why.

---

# Part 30 — Create a provenance record

```python
metadata = {
    "source": "JSONPlaceholder users endpoint",
    "url": "https://jsonplaceholder.typicode.com/users",
    "retrieved_for": "STA 556 Week 5",
    "raw_file": str(json_path),
    "processed_file": str(parquet_path)
}
```

Save:

```python
metadata_path = (
    RAW_DIR / "users_metadata.json"
)

with open(
    metadata_path,
    "w",
    encoding="utf-8"
) as f:
    json.dump(
        metadata,
        f,
        indent=2
    )
```

What else would you record for a real research dataset?

---

# Part 31 — End-to-end ingestion challenge

Build a mini ingestion pipeline that:

1. requests the data;
2. checks the HTTP response;
3. converts the response to JSON;
4. saves the raw JSON;
5. normalizes the JSON;
6. selects useful variables;
7. validates row count and IDs;
8. saves processed data to Parquet;
9. writes provenance metadata.

Skeleton:

```python
def fetch_users(
    url: str
) -> list[dict]:
    ...
```

```python
def validate_users(
    data: pd.DataFrame
) -> None:
    ...
```

```python
def save_users(
    data: pd.DataFrame,
    path: Path
) -> None:
    ...
```

---

# Part 32 — Validate the pipeline

Write checks:

```python
assert users_flat["id"].notna().all()
```

```python
assert users_flat["id"].is_unique
```

```python
assert len(users_flat) > 0
```

```python
assert "email" in users_flat.columns
```

If the API schema changed tomorrow, which assertions could alert you?

---

# Part 33 — Source-selection challenge

Choose the best approach.

### A

A public agency provides a downloadable CSV.

### B

A database contains 200 million transactions, but you need one state and one year.

### C

A website provides a documented REST API.

### D

A web page contains one clean HTML table but no downloadable file.

### E

You repeatedly store and analyze a 20 GB tabular dataset.

Possible answers:

```text
CSV
SQL query
API
read_html()
Parquet
```

Explain each choice.

---

# Part 34 — Reproducibility challenge

Compare:

### Workflow A

```text
Open website
Download file manually
Rename file
Move file to desktop
Edit several cells
Run notebook
```

### Workflow B

```text
Python downloads source
Raw source saved unchanged
Cleaning performed in code
Processed file created
Metadata recorded
Analysis reads processed file
```

Which is more reproducible?

What information is still missing even from Workflow B?

---

## Codespaces secrets for authenticated APIs

The public APIs used in this tutorial may not require authentication. If a later API requires a key, **never paste the key directly into a committed notebook or script**.

Use an environment variable or a Codespaces secret, then read it in Python:

```python
import os

api_key = os.environ[
    "MY_API_KEY"
]
```

Keep `.env` files and credentials out of Git.

---

# Part 35 — Git checkpoint

Run:

```bash
git status
```

Ask:

> Should raw data files be committed?

The answer depends on:

- size;
- licensing;
- sensitivity;
- reproducibility requirements.

Never automatically commit sensitive, proprietary, or extremely large data.

Commit appropriate code/documentation:

```bash
git add .
git commit -m "Complete Week 5 external data ingestion"
git push
```

---

# Part 36 — Final reflection

### 1. I/O

What does input/output mean in a data-science workflow?

### 2. Data types

Why inspect `dtypes` after reading a CSV?

### 3. SQL

Why might filtering in SQL before loading into pandas be preferable?

### 4. APIs

What is the relationship between an HTTP response, JSON, and a DataFrame?

### 5. JSON

Why can nested JSON require normalization?

### 6. HTML

Why might an official API be preferable to scraping?

### 7. Parquet

Why might Parquet be preferable to CSV for a large analytical dataset?

### 8. Raw data

Why should raw source data generally remain unchanged?

### 9. Provenance

What would another researcher need to determine where your data came from?

### 10. Validation

Why can ingestion fail scientifically even when no Python exception occurs?

---

# Completion checklist

- [ ] Created Week 5 notebook
- [ ] Created raw/processed/external directories
- [ ] Wrote and read CSV data
- [ ] Controlled imported data types
- [ ] Defined missing-value codes
- [ ] Validated imported CSV data
- [ ] Created an SQLite database
- [ ] Queried SQLite with SQL
- [ ] Selected and filtered using SQL
- [ ] Aggregated using SQL
- [ ] Used a parameterized query
- [ ] Performed an SQL join
- [ ] Made an HTTP request
- [ ] Checked an HTTP status code
- [ ] Used `raise_for_status()`
- [ ] Parsed JSON
- [ ] Normalized nested JSON
- [ ] Saved a raw API response
- [ ] Read a local JSON snapshot
- [ ] Explored an HTTP error
- [ ] Used `pd.read_html()`
- [ ] Saved processed data
- [ ] Wrote and read Parquet
- [ ] Compared CSV and Parquet
- [ ] Explored HDF5 or documented its dependency
- [ ] Created provenance metadata
- [ ] Built an end-to-end ingestion pipeline
- [ ] Added validation checks
- [ ] Completed the source-selection challenge
- [ ] Committed appropriate Week 5 work to Git
- [ ] Pushed work to GitHub

---

# What you should now understand

```text
External data source
        ↓
Choose access method
        ↓
Read / query / request
        ↓
Check success
        ↓
Parse structure
        ↓
Validate schema + values
        ↓
Preserve raw input
        ↓
Transform
        ↓
Record provenance
        ↓
Save analysis-ready data
```

Next week we move into **Exploratory Data Analysis and Visualization**.
