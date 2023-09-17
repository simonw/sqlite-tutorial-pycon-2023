# sqlite-utils

[sqlite-utils](https://sqlite-utils.datasette.io/) provides "CLI tool and Python utility functions for manipulating SQLite databases".

You can install it the same way as Datasette:

    pip install sqlite-utils

Or with `pipx`:

    pipx install sqlite-utils

Or with Homebrew:

    brew install sqlite-utils

It works as both a CLI tool and a Python library.

## Using the command-line tools to clean data

We'll follow this tutorial next: **[Cleaning data with sqlite-utils and Datasette](https://datasette.io/tutorials/clean-data)**

## Using sqlite-utils as a Python library, to import all the PEPs

Let's take our PEPs example from earlier and implement it again, but better, using `sqlite-utils`.

I'll do this in a notebook.

```
!git clone https://github.com/python/peps /tmp/peps
```
We now have ALL of the PEPs in `/tmp/peps`

```python
import pathlib

files = list(pathlib.Path("/tmp/peps/peps").glob("pep-*.rst"))
```
And parse them with our function from earlier:
```python
def parse_pep(s):
    intro, body = s.split("\n\n", 1)
    pep = {}
    current_key = None
    current_value = None
    for line in intro.split("\n"):
        # If the line starts with whitespace, it's a continuation of the previous value
        if line.startswith(" ") or line.startswith("\t"):
            if current_key is not None:
                current_value += " " + line.strip()
                pep[current_key] = current_value.strip()
        else:
            # Split the line into key and value
            parts = line.split(": ", 1)
            if len(parts) == 2:
                key, value = parts
                # Update the current key and value
                current_key = key
                current_value = value
                # Add the key-value pair to the pep dictionary
                pep[current_key] = current_value.strip()
    pep["Body"] = body.strip()
    return pep
```
```python
peps = []
for file in files:
    peps.append(parse_pep(file.read_text()))
```
We now have a list of dictionaries. Let's load them into SQLite:
```
%pip install sqlite-utils
```
```python
import sqlite_utils
db = sqlite_utils.Database("/tmp/peps.db")
db["peps"].insert_all(peps, pk="PEP", alter=True, replace=True)
```
I got this error:
```
OperationalError: table peps has no column named PEP-Delegate
```
To fix that:
```
db["peps"].insert_all(peps, pk="PEP", alter=True, replace=True)
print(db["peps"].count)
# Outputs 429 
```
## Enabling full-text search

SQLite has surprisingly good full-text search built in.

`sqlite-utils` can help you enable it:

```python
db["peps"].enable_fts(["Title", "Body"])
```
Datasette will detect this and add a search box to the top of the table page.

To run searches in relevance order you'll need to execute a custom SQL query:

```sql
select
  PEP,
  peps.Title,
  Version,
  Author,
  Status,
  Type,
  Created,
  peps.Body,
  peps_fts.rank
from
  peps
join
  peps_fts on peps.rowid = peps_fts.rowid
where
  peps_fts match :search
order by
  peps_fts.rank
limit
  20
```
