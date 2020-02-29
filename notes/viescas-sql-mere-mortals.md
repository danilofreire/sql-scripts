# Notes on "John Viescas -- SQL Queries for Mere Mortals" (2018)

## Part I - Relational Databases and SQL

### Chapter 01 - "What Is Relational?"

Some basic definitions:

* **Relational Database Management System (RDBMS)**: an application to create,
  modify, and manipulate a database.
* **Relation**: part of [set theory](https://en.wikipedia.org/wiki/Set_theory),
  expressed by a table.
* **Tuples**: rows.
* **Attributes**: columns.
* **Primary keys**: one or more columns that provide unique IDs for the rows.
* **Foreign keys**: one or more primary keys from another table which when inserted
  into a second table becomes a new key. Establish the relationship between two
  or more tables and avoids "orphaned rows" without reference.
* **Views**: a visual table with your queries. The information you've selected. 
* Relationships: 
  - *One-to-One*: a single row in table A is related to one and only one row in
	table B.
  - *One-to-Many*: a single row in table A is related to many rows in table B. 
  - *Many-to-Many*: many rows in table A are related to many rows in table B. You
	must create a _linking table_: a copy of the primary key of each table that
	can be combined together to make a composite primary key.

### Chapter 02 - "Ensuring Your Database Structure Is Sound"

It is important to certify the integrity of your dataset. The author gives some
tips:

* **Names**: choose a name that is clear to everyone; be specific; don't use the
same name for other columns (except for keys); don't use acronyms; and only use
the singular form of each name.
* **Multipart columns (columns with more than one information)**: break them into
the smaller parts possible. For instance, if you have a column with full name,
break it into first and last name. Address? Divide it into street name, number,
zip code, city, and state.
* **Multivalue columns**: can be hard to solve. Usually they contain data that 
are separated by commas or semicolons. You can use a linking table as multivalue
columns have many-to-many relationships. Use the combination of variables as a
new composite primary key and they will be associated in a one-to-one basis.
* **Unnecessary columns**: if there's a key that identifies the same people in
two tables, delete the duplicated attributes in one of the tables (apart from
the key). If you have one table, create a composite primary key that identifies
a unique row and delete the rest. Finally, if the columns have missing data,
try to find one that doesn't and use that one. Delete the others.

----

* **Primary keys**: should be unique, without missing observations, without
multivalues, and must never be modified. These are _necessary_ conditions. If
these conditions are not met, create an arbitrary column with IDs. But first
check if there isn't other group of variables that can uniquely identify each
row in the data itself.

----

How to delete rows? There are two rules:

* **Restrict deletion rule**: you must delete any row in the subordinate
tables before deleting the row in the primary table. This is the most common
case.
* **Cascade delete rule**: this rule deletes the _n_ row in all databases if
the row is deleted in one of them. Avoid it.

----

* **Degrees of participation**: the number of rows one dataset is allowed to
have in a joined table. It varies according to your requests.

### Chapter 03: "A Concise History of SQL"

No notes are necessary.

## Part II - SQL Basics

### Chapter 04: "Creating a Simple Query"

The workhorse of SQL is the `SELECT` command. It answers "nearly any question
"regarding who, what, where, when or even what if", as the author writes. He
breaks the `SELECT` command into three functions.

* **`SELECT` statement**: simply a query. The statement can have many clauses,
which the main are:
  - `FROM`: the table you are querying from. Required.
  - `WHERE`: optional clause to filter the results.
  - `GROUP BY`: divides the information into groups. Optional too.
  - `HAVING`: similar to `WHERE` but for aggregate information. Used after
	`GROUP BY`.
  - `ORDER BY`: sorts the results of the query. Doesn't affect the order of the
	rows in the original table. Use `DESC` for descending order, `ASC` for
	ascending order.
  - Example: `SELECT var1, var2 FROM df01 ORDER BY var3 ASC, var4 DESC`; `SELECT * FROM df02`, where `*`
	means all columns.
  - `SELECT DISTINCT var` will delete all duplicated entries.
  - `SELECT TOP N`: where `N` is the number of rows. Like `head()` in `R` or
	`Python`.

### Chapter 05: "Getting More Than Simple Columns" 

SQL has seven data types: 

* `CHAR`: saves character values. If you want to specify how many characters
  (maybe to save space), type `VCHAR (n)`. Uses characters from the English
  language. For longer texts, type `TEXT`.
* `NCHAR`: national characters. Use ISO symbols, so it includes more characters
  than those of the English language, such as Japanese. `NTEXT` is the large
  version.
* `BIT`: store video, audio, and other complex data.
* `NUMERIC, DECIMAL, DEC, SMALLINT, INTEGER, INT, BIGINT`: all refer to exact
  numeric data.
* `FLOAT`: approximate numeric. 
* `BIT, INT, TINYINT`: Boolean (true or false).
* `DATE, TIME, DATETIME`: the first two are self-explanatory, the last one
  stores both together.  
* `INTERVAL`: difference in time or date between two variables. Not every
  version of SQL uses it.

The `CAST` function set the type of variable. `CAST (var01 AS INT)`

----

Quotes are used for: 

* **String variables**: like in `'This is a string'`. 
* **Numeric literals**: such as `'42'`. 
* **Datetime literals**: date, time, datetime literals such as `'2020-10-02'`,
  `'17:45:23'`, or `'2001-11-12 13:23:21'`. 
