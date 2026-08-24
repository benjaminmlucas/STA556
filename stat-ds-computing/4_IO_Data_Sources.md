# 2.3 Input/Output & External Data Sources

## Why this week matters

In Weeks 3 and 4, we worked with DataFrames that already existed inside Python. Real data science rarely begins that way.

Data may live in:

- CSV files;
- relational databases;
- web APIs;
- JSON responses;
- HTML tables;
- Parquet files;
- HDF5 files;
- remote servers.

The central question for Week 5 is:

> **How do we reliably move data from external systems into a statistical computing workflow?**

The approved course schedule identifies Week 5 as **I/O and External Data Sources**, including:

- SQL databases;
- web APIs;
- JSON and HTML data;
- web scraping;
- binary file formats such as Parquet and HDF5.

The emphasis this week is not simply on knowing several `pandas.read_*()` functions. It is on understanding where data come from, what structure they have, how to ingest them correctly, how to validate them after ingestion, and how source format affects reproducibility and efficiency.

---

# 1. What is I/O?

**I/O** means **input/output**.

```text
External source
      ↓
    INPUT
      ↓
    Python
      ↓
Transformation / analysis
      ↓
    OUTPUT
      ↓
File / database / report
```

Common input functions:

```python
pd.read_csv(...)
pd.read_sql(...)
pd.read_json(...)
pd.read_html(...)
pd.read_parquet(...)
```

Common output functions:

```python
df.to_csv(...)
df.to_sql(...)
df.to_json(...)
df.to_parquet(...)
```

> **A data-science workflow needs a reproducible boundary between external data and internal analysis.**

---

# 2. File formats encode structure

Different formats represent data differently.

```text
CSV      → plain-text rectangular table
JSON     → nested objects / arrays
HTML     → web document, possibly containing tables
SQL      → relational tables queried from a database
Parquet  → columnar binary tabular format
HDF5     → hierarchical binary data container
```

Format affects storage size, speed, data types, nested structure, interoperability, and reproducibility.

---

# 3. CSV as a baseline

A CSV file might look like:

```text
id,age,group,score
101,24,A,81
102,31,B,94
103,27,A,88
```

Read it:

```python
import pandas as pd

df = pd.read_csv("data/participants.csv")
```

Inspect immediately:

```python
df.head()
df.shape
df.dtypes
df.info()
```

> Reading a file successfully does not prove that pandas interpreted it correctly.

---

# 4. Common CSV ingestion problems

Potential problems include:

- wrong delimiter;
- header incorrectly interpreted;
- dates imported as strings;
- IDs imported as numbers;
- missing-value codes such as `-999`;
- inconsistent encodings;
- numeric values containing commas;
- unexpected extra columns.

Examples:

```python
df = pd.read_csv(
    "data/data.csv",
    na_values=["NA", "N/A", -999]
)
```

```python
df = pd.read_csv(
    "data/data.csv",
    parse_dates=["date"]
)
```

Ingestion requires decisions.

---

# 5. Read only what you need

For large files:

```python
df = pd.read_csv(
    "large.csv",
    usecols=["id", "age", "score"]
)
```

Specify types where appropriate:

```python
df = pd.read_csv(
    "large.csv",
    dtype={
        "id": "string",
        "group": "category"
    }
)
```

This can reduce memory use and prevent incorrect type inference.

---

# 6. Paths and reproducibility

## Codespaces path convention

In the course Codespace, the repository is under `/workspaces/<repository-name>/`, but analysis code should normally **not hard-code that absolute location**. Work from the repository root and use relative paths.

Avoid computer-specific paths such as:

```python
pd.read_csv(
    "/Users/alex/Desktop/STA556/data.csv"
)
```

or Windows-specific equivalents such as `C:\\Users\\...`.

Prefer:

```python
pd.read_csv(
    "data/data.csv"
)
```

or:

```python
from pathlib import Path

DATA_DIR = Path("data")

df = pd.read_csv(
    DATA_DIR / "data.csv"
)
```

This connects directly to Week 1's project-structure principles.

---

# 7. Relational databases

A relational database stores data in tables.

```text
participants
────────────
id
age
group

scores
──────
id
score
```

Tables can be related through keys.

Rather than loading every record and filtering afterward, SQL lets us ask the database for the data we need.

---

# 8. SQL

SQL stands for **Structured Query Language**.

```sql
SELECT *
FROM participants;
```

Choose variables:

```sql
SELECT id, age, score
FROM participants;
```

Filter:

```sql
SELECT id, age, score
FROM participants
WHERE age >= 30;
```

SQL therefore performs operations conceptually similar to pandas selection and filtering.

---

# 9. SQL and pandas

Python's built-in `sqlite3` module can work with SQLite databases.

```python
import sqlite3

connection = sqlite3.connect(
    "data/study.db"
)
```

Then:

```python
query = """
SELECT *
FROM participants
"""

df = pd.read_sql(
    query,
    connection
)
```

This creates a DataFrame from a database query.

---

# 10. Push filtering toward the data source

Suppose a database contains ten million records, but only 1% are needed.

Less efficient:

```python
df = pd.read_sql(
    "SELECT * FROM observations",
    connection
)

df = df.loc[df["group"] == "A"]
```

Often better:

```python
query = """
SELECT *
FROM observations
WHERE group = 'A'
"""

df = pd.read_sql(
    query,
    connection
)
```

Why move more data than necessary?

---

# 11. SQL joins

The join ideas from Week 4 also exist in SQL.

```sql
SELECT
    p.id,
    p.age,
    p.group,
    s.score
FROM participants AS p
LEFT JOIN scores AS s
    ON p.id = s.id;
```

Compare with:

```python
participants.merge(
    scores,
    on="id",
    how="left"
)
```

The syntax differs, but the relational logic is the same.

---

# 12. Parameterized SQL queries

Avoid constructing SQL by directly inserting values into strings.

Prefer parameterized queries:

```python
query = """
SELECT *
FROM participants
WHERE group = ?
"""

df = pd.read_sql(
    query,
    connection,
    params=("A",)
)
```

This separates query logic from parameter values and is safer.

---

# 13. What is an API?

An **Application Programming Interface (API)** provides a structured way for software systems to communicate.

For data science, APIs often provide data over HTTP.

```text
Python
   │
   │ HTTP request
   ▼
Web API
   │
   │ response
   ▼
JSON data
```

Instead of downloading a static file manually, our program can request data directly.

---

# 14. HTTP requests

The `requests` library is commonly used for HTTP requests.

```python
import requests

response = requests.get(
    "https://example.com/api/data"
)
```

Inspect:

```python
response.status_code
```

A successful request commonly returns `200`.

Use:

```python
response.raise_for_status()
```

to raise an exception for unsuccessful responses.

---

# 15. API parameters

APIs frequently accept query parameters.

```python
params = {
    "state": "AZ",
    "year": 2025
}

response = requests.get(
    url,
    params=params
)
```

This is preferable to manually constructing long URLs because parameters remain explicit.

---

# 16. JSON

JSON stands for **JavaScript Object Notation**.

```json
{
  "id": 101,
  "age": 24,
  "group": "A"
}
```

JSON maps naturally onto Python:

```text
JSON object → dict
JSON array  → list
string      → str
number      → int / float
true/false  → True / False
null        → None
```

This connects directly to Week 2.

---

# 17. Parse JSON from an API

```python
response = requests.get(url)

response.raise_for_status()

data = response.json()
```

Inspect:

```python
type(data)
```

If the response is a list of dictionaries:

```python
df = pd.DataFrame(data)
```

But many APIs return nested structures, so inspect before conversion.

---

# 18. Nested JSON

Consider:

```python
data = {
    "results": [
        {
            "id": 101,
            "location": {
                "state": "AZ",
                "city": "Flagstaff"
            }
        }
    ]
}
```

Use:

```python
pd.json_normalize(
    data["results"]
)
```

Result:

```text
id   location.state   location.city
101  AZ               Flagstaff
```

Nested structures often need normalization before analysis.

---

# 19. APIs may be paginated

An API might return only 100 observations per request.

```text
page 1
page 2
page 3
...
```

Conceptually:

```python
records = []

for page in pages:
    response = ...
    records.extend(...)
```

Ask:

- How many records per page?
- How do we detect the final page?
- Is there a rate limit?
- Is authentication required?

---

# 20. API reliability

External systems fail.

Possible problems:

```text
404 → resource not found
429 → too many requests
500 → server error
timeout
network failure
invalid JSON
```

A better request includes a timeout:

```python
response = requests.get(
    url,
    timeout=10
)

response.raise_for_status()
```

Later in STA 556, exception handling will make these workflows more robust.

---

# 21. HTML data

Sometimes data are embedded in web pages.

```python
tables = pd.read_html(
    "https://example.com/page"
)
```

`read_html()` returns a **list of DataFrames**.

Inspect:

```python
len(tables)
tables[0]
```

Do not assume the first table is necessarily the one you need.

---

# 22. HTML tables vs. web scraping

### HTML table ingestion

```python
pd.read_html(...)
```

is appropriate when data already exist in HTML table elements.

### Web scraping

General scraping extracts information from arbitrary page structure, often with tools such as BeautifulSoup.

Scraping can be fragile because website structure changes.

Whenever a stable official API or downloadable source exists, it is often preferable.

---

# 23. Responsible web data collection

Before scraping or repeatedly requesting a site:

- check whether an official API exists;
- inspect terms of use;
- respect rate limits;
- avoid excessive requests;
- cache downloaded data where appropriate;
- record the source and retrieval date.

Technical ability does not imply unrestricted permission.

---

# 24. Binary formats

CSV is highly portable, but it does not naturally preserve rich type information or provide efficient columnar storage.

Two binary formats listed in the course syllabus are:

```text
Parquet
HDF5
```

---

# 25. Parquet

Parquet is a **columnar storage format** widely used in analytics.

Write:

```python
df.to_parquet(
    "data/participants.parquet"
)
```

Read:

```python
df = pd.read_parquet(
    "data/participants.parquet"
)
```

Typical advantages include:

- compression;
- fast analytical reads;
- better type preservation;
- efficient column selection.

---

# 26. Columnar storage

A CSV conceptually stores rows:

```text
101,24,A,81
102,31,B,94
```

A columnar format organizes values primarily by column.

For analytics such as:

```python
df[["age", "score"]]
```

a columnar format can avoid reading unrelated columns.

---

# 27. HDF5

HDF5 is a hierarchical binary format for large and complex scientific datasets.

```python
df.to_hdf(
    "data/study.h5",
    key="participants"
)
```

Read:

```python
df = pd.read_hdf(
    "data/study.h5",
    key="participants"
)
```

A single HDF5 file can contain multiple named objects.

---

# 28. CSV vs. Parquet

| Property | CSV | Parquet |
|---|---|---|
| Human-readable | Yes | No |
| Preserves types well | Limited | Yes |
| Compression | Optional/external | Built in |
| Columnar | No | Yes |
| Interoperability | Extremely high | High |
| Large analytic datasets | Less efficient | Often more efficient |

The appropriate choice depends on the workflow.

---

# 29. Never trust ingestion blindly

After reading external data:

```python
df.shape
df.head()
df.tail()
df.dtypes
df.isna().sum()
```

Inspect important variables:

```python
df["id"].nunique()
df["group"].value_counts()
```

Write checks:

```python
assert df["id"].notna().all()
```

```python
assert df["age"].between(
    0, 120
).all()
```

---

# 30. Preserve raw data

A useful structure is:

```text
data/
├── raw/
├── processed/
└── external/
```

A common principle:

> **Do not manually edit the raw source data.**

Instead:

```text
raw data
   ↓
Python transformation
   ↓
processed data
```

---

# 31. Cache external data

If every notebook run calls an API again:

- the data may change;
- the API may become unavailable;
- the request may be slow;
- rate limits may be triggered.

A useful pattern is:

```text
API
 ↓
download once
 ↓
save raw response
 ↓
analysis reads local snapshot
```

---

# 32. Record provenance

For external data, record:

```text
Source
URL / endpoint
Retrieval date
Query parameters
File format
Data version
Transformation steps
```

Data provenance answers:

> **Where did these data actually come from?**

---

# 33. Separate ingestion from analysis

Avoid a single notebook that downloads, cleans, models, and plots everything.

Instead:

```text
01_ingest.py
      ↓
raw data

02_clean.py
      ↓
processed data

03_analysis.ipynb
      ↓
results
```

Separating stages makes workflows easier to debug and reproduce.

---

# 34. Key ideas

By the end of Week 5, you should be able to explain:

1. What input/output means in a data workflow.
2. Why source format affects data handling.
3. How to ingest CSV data safely.
4. Why project-relative paths matter.
5. What a relational database is.
6. How SQL relates to pandas operations.
7. How to use `pd.read_sql()`.
8. What an API is.
9. How HTTP requests and status codes work.
10. How JSON maps onto Python objects.
11. How to normalize nested JSON.
12. Why APIs may require pagination and error handling.
13. How `pd.read_html()` differs from general scraping.
14. Why responsible web data collection matters.
15. Why Parquet can outperform CSV for analytical workloads.
16. What HDF5 is used for.
17. Why ingestion must be validated.
18. Why raw data should be preserved.
19. Why caching external data improves reproducibility.
20. What data provenance means.

---

# 35. Recommended reading

## pandas — IO tools

https://pandas.pydata.org/docs/user_guide/io.html

## pandas — SQL queries

https://pandas.pydata.org/docs/reference/api/pandas.read_sql.html

## Requests — Quickstart

https://requests.readthedocs.io/en/latest/user/quickstart/

## Python `sqlite3`

https://docs.python.org/3/library/sqlite3.html

## Python for Data Analysis — Wes McKinney

https://wesmckinney.com/book/

---

# 36. YouTube recommendations

## 1. Data Geek is my name — "SQLite + Pandas — Learn SQL with Python for Beginners in 10 mins"

A concise demonstration of creating and querying an SQLite database with Python and loading SQL results into pandas DataFrames.

**Recommended use:** Watch before the SQL portion of the hands-on tutorial.

[Watch on YouTube](https://www.youtube.com/watch?v=VAwDZRBXb8w)

---

## 2. Lab Time with R & Python — "Python: Retrieve Data from API using Requests | JSON to pandas data frame"

A practical demonstration of making API requests with Python, receiving JSON responses, and converting simple and nested responses into pandas DataFrames.

**Recommended use:** Watch alongside the API and JSON sections.

[Watch on YouTube](https://www.youtube.com/watch?v=AjThw99ZhMk)

---

## 3. freeCodeCamp.org — APIs for Beginners

A broader introduction to APIs, endpoints, requests, responses, authentication, and HTTP concepts.

**Recommended use:** Optional extension for students who want more conceptual background on APIs.

[Find on YouTube](https://www.youtube.com/results?search_query=freeCodeCamp+APIs+for+Beginners)

---

# 37. Week 5 takeaway

> **External data should enter an analysis through a documented, validated, and reproducible ingestion process.**

```text
External source
       ↓
Understand format
       ↓
Retrieve / query
       ↓
Parse
       ↓
Validate
       ↓
Preserve raw data
       ↓
Transform
       ↓
Record provenance
       ↓
Analysis-ready data
```

Next week we move into **Exploratory Data Analysis and Visualization**.
