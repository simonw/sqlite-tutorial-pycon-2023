# Introduction to SQLite

## Why SQLite?

- It's the database you already have - `sqlite3` has been built into Python since 2006
- It's screamingly fast, and surprisingly powerful
- Amazing compatibility: bindings for every language, files work on every platform, fantastic track record for backwards compatibility, so it's safe to trust your data to a SQLite file
- Databases are just files on disk. You can create and discard them without any ceremony.
- It handles text (including JSON), integers, floating point numbers, and binary blobs. Which means it can store _anything_.
- It can handle up to 2.8TB of data(!)
- It has some interesting characteristics, for example [Many Small Queries Are Efficient In SQLite](https://www.sqlite.org/np1queryprob.html)

## First steps with Python

Let's download a database to play with - we'll use the database that powers the <https://datasette.io/> website:

```
wget https://datasette.io/content.db
```
To access it from Python:
```python
import sqlite3

db = sqlite3.connect("content.db")

print(db.execute("select sqlite_version()").fetchall())
# [('3.39.0',)]

# Show rows from the plugin_repos table
for row in db.execute("SELECT * FROM plugin_repos LIMIT 10"):
    print(row)

# Each row is a tuple. We can change that like this:
db.row_factory = sqlite3.Row

for row in db.execute("SELECT * FROM plugin_repos LIMIT 10"):
    print(row)

# This outputs <sqlite3.Row object at 0x7f5d3d8a3760>
# We can use dict() to turn those into dictionaries instead
for row in db.execute("SELECT * FROM plugin_repos LIMIT 10"):
    print(dict(row))

```

## Creating a table

Let's create a table:
```python
db.execute("""
create table peps (
  id integer primary key,
  title text,
  author text,
  status text,
  type text,
  created text,
  body text
);
""")
```

## Inserting some data

Here's a function I wrote that can parse a PEP:

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
Let's fetch and parse the Zen of Python:
```python
import urllib

zen = urllib.request.urlopen(
    "https://raw.githubusercontent.com/python/peps/main/pep-0020.txt"
).read().decode("utf-8")

pep = parse_pep(zen)
```
And insert that into our database:
```python
db.execute("""
    insert into peps (
        id, title, author, status, type, created, body
    ) values (
        ?, ?, ?, ?, ?, ?, ?
    )
""", (
    pep["PEP"],
    pep["Title"],
    pep["Author"],
    pep["Status"],
    pep["Type"],
    pep["Created"],
    pep["Body"],
))
```
Since this is a dictionary already, we can use alternative syntax like this:
```python
db.execute("delete from peps where id = 20")
db.execute("""
    insert into peps (
        id, title, author, status, type, created, body
    ) values (
        :PEP, :Title, :Author, :Status, :Type, :Created, :Body
    )
""", pep)
```
To confirm that it was correctly inserted:
```python
print(db.execute("select * from peps").fetchall())
```

## UPDATE and DELETE

To update a record:

```python
with db:
    db.execute("""
    update peps set author = ?
    where id = ?
    """, ["Tim Peters", 20])
```
This will run in a transaction.

To delete a record:

```python
with db:
    db.execute("""
    delete from peps
    where id = ?
    """, [20])
```
Or to delete everything:
```sql
delete from peps
```
## SQLite column types

SQLite `create table` is easier than many other databases, because there are less types to worry about. There are four types you need to worry about:

- `integer`
- `real`
- `text`
- `blob`

Unlike other databases, length limits are neither required or enforced - so don't worry about `varchar(255)`, just use `text`.

Tables automatically get an ID column called `rowid` - an incrementing integer. This will be the primary key if you don't specify one.

If you specify `integer primary key` it will be auto-incrementing and will actually map to that underlying `rowid`.

You can set `id text primary key` for a text primary key - this will not increment, you will have to set it to a unique value for each row yourself. You could do this with UUIDs generated using `uuid.uuid4()` for example.

SQLite is loosely typed by default: you can insert any type into any column, even if it conflicts with the column type!

A lot of people find this very uncomfortable.

As-of SQLite 3.37.0 (2021-11-27) you can set [strict mode](https://www.sqlite.org/stricttables.html) on a table to opt-out of this loose typing:

```sql
create table peps (
  id integer primary key,
  title text,
  author text,
  body text
) strict
```

## Transactions

Here's an example of the impact transactions have on file-based databases:

```pycon
>>> import sqlite3
>>> db = sqlite3.connect("/tmp/data.db")
>>> db.execute("create table foo (id integer primary key, name text)")
<sqlite3.Cursor object at 0x102ec5c40>
```
In another window:
```
% sqlite3 data.db .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE foo (id integer primary key, name text);
COMMIT;
```
```pycon
>>> db.execute('insert into foo (name) values (?)', ['text'])
<sqlite3.Cursor object at 0x102ec5bc0>
```
In the other window:
```
% sqlite3 data.db .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE foo (id integer primary key, name text);
COMMIT;
```
```pycon
>>> db.commit()
```
And now:
```
% sqlite3 data.db .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE foo (id integer primary key, name text);
INSERT INTO foo VALUES(1,'text');
COMMIT;
```
A nicer pattern is to do this:
```pycon
>>> with db:
...     db.execute('insert into foo (name) values (?)', ['text'])
```
The `with db:` wraps everything inside that block in a transaction.
