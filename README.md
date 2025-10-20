# Getting Started with DuckDB

This is a quick introduction to [DuckDB][]. They have excellent documentation,
so be sure to check out their website for more information!

[DuckDB]: https://duckdb.org/


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

To demo DuckDB, I'll use a [Pixi][] environment with the `python`, `jupyter`,
`pandas`, `polars`, and `python-duckdb` packages. See the `pixi.toml` file in
this repository for the exact specification.

[Pixi]: https://pixi.sh/

The DuckDB documentation has [a section specifically about the Python
API][duckdb-py].

[duckdb-py]: https://duckdb.org/docs/stable/clients/python/overview
