# Getting Started with DuckDB

This is a quick introduction to [DuckDB][]. They have excellent documentation,
so be sure to check out their website for more information!

[DuckDB]: https://duckdb.org/

This mini-workshop assumes you're either already familiar with SQL or
comfortable with seeing some SQL code that you might not fully understand yet.


## What is DuckDB?

DuckDB is a **database system**: software to help you store, organize, and
query data.

Like many database systems, DuckDB:

* Is especially good at working with **relational data**, data that consist of
  multiple tables of related data that can be linked together by columns they
  have in common.
* Uses **structured query language** (SQL or "sequel") as the primary way to
  describe what you want to do with data.

Distinguishing features of DuckDB include that:

* It's free and open-source.
* It has no external dependencies and runs on all major operating systems and
  hardware architectures.
* It supports reading from and writing to many different data formats, not just
  its own database format.
* A DuckDB database is just a file.
* It's designed from the ground up for data analysis tasks. Most popular
  databases (such as SQLite, PostgreSQL, MariaDB) are optimized for online
  transaction processing (OLTP) workloads, where data are frequently added or
  updated, but most queries are short and simple. In contrast, DuckDB is
  optimized for **online analytical processing** (OLAP) workloads, where data
  change infrequently, but queries are often long or complex.
  

## When Should I Use DuckDB?

DuckDB is great for analyzing tabular or relational data, and as a format for
storing data. I've specifically found it helpful for working with:

* Data that will grow over time. I frequently use [Parquet][] files because
  Parquet is a widely-supported open standard and the files are fast,
  compressed, and preserve the data types of columns. The disadvantages of
  Parquet are that each file can only store one table and that it doesn't
  provide an efficient way to append data (which is important if, for example,
  the data will be scraped from the web). DuckDB offers most of the advantages
  of Parquet (DuckDB is faster, but the compression is worse) without these two
  disadvantages.
* Data that needs to be accessible to many different collaborators. DuckDB
  provides packages for Python, R, and many other programming languages. For
  macOS and Linux, there's also a command-line interface you can use to run SQL
  queries directly.
* Naturally relational data. Put your tables in a DuckDB database and you can
  easily join them on shared columns. This isn't a unique feature of DuckDB,
  but DuckDB is easy to install and can import data from many formats,
  including CSV, Parquet, and other databases.

[Parquet]: https://parquet.apache.org/


## How Do I Use DuckDB with Python?

### Setup

To demo DuckDB, I'll use a [Pixi][] environment with the `python`, `jupyter`,
`pandas`, `pyarrow`, `polars`, and `python-duckdb` packages. See the
`pixi.toml` file in this repository for the exact specification. To use the
environment, first install Pixi (if you haven't already). Then open a terminal,
go to the top level of this repository, and run:

`pixi shell`

[Pixi]: https://pixi.sh/

We'll also use these datasets:

* [2000-2023 California Least Tern dataset][tern-data] ([more info][tern-info])
* [2024 Sacramento & Yolo Crash dataset][crash-data] [[more info][crash-info])

[tern-data]: https://ucdavis.box.com/s/m2w5l2ebp2rey2do5lnn38y1e3u31pin
[tern-info]: https://ucdavisdatalab.github.io/workshop_python_basics/chapters/01_python-basics.html#hello-data
[crash-data]: https://ucdavis.box.com/s/kjxowylvg3cfqgnofh03gn5e3xicsobt
[crash-info]: https://ucdavisdatalab.github.io/workshop_intermediate_python/chapters/02_tidy-relational-data.html#case-study-ca-crash-reporting-system

The DuckDB documentation has [a section specifically about the Python
API][duckdb-py].

[duckdb-py]: https://duckdb.org/docs/stable/clients/python/overview


### Importing Data from Files

DuckDB defaults to storing databases in your computer's memory (rather than in
a file). Let's read the CA least tern dataset into DuckDB's in-memory database.
Open Python, IPython, or Jupyter, and run:

```python
import duckdb

terns = duckdb.read_csv("data/2000-2023_ca_least_tern.csv")
```

The `duckdb.read_csv` functions and most other functions and methods that read
or query data return a `DuckDBPyRelation` object, which represents a table in
the database.  I'll call these tables or **relations**. There are
`duckdb.read_` functions for many common file formats.


### Running Queries

SQL queries are the primary way to work with data in DuckDB, although the
Python API also provides methods for common tasks. Use `duckdb.sql` to run a
query against the current database. DuckDB automatically keeps track of
(Python) variables that refer to DuckDB objects, so you can use the variables
in your SQL queries. Here's a query that gets all rows where `year` is `2000`
in the `terns` data:

```python
y2k = duckdb.sql("SELECT * FROM terns WHERE year = 2000")
```

By naming a result (in this case, `y2k`), you can run additional queries on it:

```python
duckdb.sql("SELECT site_name FROM y2k LIMIT 5")
```


### Importing Data from Python

DuckDB keeps track of (Python) variables that refer to Pandas or Polars data
frames, in the same way that it does for variables that refer to DuckDB
objects. So you can use `duckdb.sql` to run a query on a data frame:

```python
import polars as pl

toy_data = pl.DataFrame({"x": [1, 2, 3], "y": [2000, 2000, 2001]})

duckdb.sql("SELECT * FROM toy_data WHERE y = 2000")
```


### Exporting Data to Python

Sometimes you might need to get data out of DuckDB for use in other Python
code. The `.fetchone` method on relations returns one row as a tuple:

```python
y2k.fetchone()
```

Similarly, the `.fetchall` method returns all rows as a list of tuples.

Since DuckDB tables are tables, you might prefer to get them out of DuckDB as
Pandas or Polars data frames. You can convert a relation to a Pandas data frame
with the `.df` method:

```python
y2k.df()
```

You can convert a relation to a Polars data frame with the `.pl` method:

```python
y2k.pl()
```


### Exporting Data to Files

DuckDB relations have `.write_` methods to write them to specific file formats.
For instance, you can use `.write_csv` to write a CSV file.


### Persistent DuckDB Databases

By default, DuckDB databases exist in-memory and disappear as soon as you end
your session (that is, close Python). Sometimes you might want to save the
entire database to a file, so that you can close it and later reopen it. To do
this, open a connection to a file (it doesn't have to exist yet) at the
beginning of your session:

```python
import duckdb

with duckdb.connect("path/to/file.db") as con:
    # Use `con` in the same way as we used `duckdb` before.
    con.sql("SELECT * FROM my_table")
```

The official documentation recommends extension `.db` or `.duckdb` for DuckDB
files.

DuckDB files have a locking mechanism so that only one process or user can open
the file at a time. If you need a multi-user database, use something like
[PostgreSQL][] instead.

[PostgreSQL]: https://www.postgresql.org/


## What Else Can DuckDB Do?

DuckDB has many more features. Here are a few relevant to researchers:

* R and Julia packages, so you can use your DuckDB databases with those
  languages too (among many others).
* Support for many different extensions. For example, there's a geospatial
  extension to make it possible to work with (vector) geospatial data.
* ["Friendly SQL"][friendly-sql], optional syntax in DuckDB's SQL dialect
  intended to make it easier to use, type, and read.
* [DuckLake][], a system and format for storing and cataloging multiple
  datasets, possibly with multiple versions (a generic name for this kind of
  tool is a **data lake**).

[friendly-sql]: https://duckdb.org/docs/stable/sql/dialect/friendly_sql
[DuckLake]: https://ducklake.select/

Check out the DuckDB website and documentation for more!
