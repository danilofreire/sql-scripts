# Introduction to Relational Databases

## Information Schema

`information_schema` is a meta-database which contains information about the current database.

## Altering Tables

The commands `ALTER TABLE` and `ADD COLUMN` allows us to add a column to an existing table (oh, seriously?!). Let's see how it works:

```sql
CREATE TABLE df01 (var01 numeric, var02 int)
ALTER TABLE df01
ADD COLUMN var03 text;
SELECT *
FROM df01
```

You can use `INSERT INTO` to add existing data to a new table.

```sql
INSERT INTO df02
SELECT DISTINCT names, titles
FROM df01;
```

`RENAME COLUMN` does exactly what it says.

```sql
ALTER TABLE df01
RENAME COLUMN names TO full_names;
```

Can you guess what `DROP COLUMN` does? I'm sure you do!

```sql
ALTER TABLE df01
DROP COLUMN full_names;
```

## Integrity Constraints

Constraints give structure to data. For instance, data types restrict operations that can be done with the data. `CAST` allows you to change the data type.

```sql
SELECT names, titles, CAST(year AS INT) - birth_year AS age
FROM dataset;
```

`ALTER TABLE` plus `ALTER COLUMN` plus `TYPE` work fine too.

```sql
ALTER TABLE dataset
ALTER COLUMN names
TYPE text;

ALTER TABLE
ALTER COLUMN weight
TYPE integer
USING ROUND(weight);
```

If the string line is too long, you can change it with `USING SUBSTRING`.

```sql
ALTER TABLE data
ALTER COLUMN names
TYPE varchar(8)
USING SUBSTRING(names FROM 1 FOR 8);
```

Add a `SET NOT NULL` constraint to data to remove null observations. `DROP NOT NULL` will remove the constraint.

```sql
CREATE TABLE table1(var01 int not null);

ALTER TABLE data
ALTER COLUMN names
SET NOT NULL;
```

Adding `UNIQUE` constraints is easy too. Curiously, to add this constraint after you created a table, use `ADD CONSTRAINT`.

```sql
CREATE TABLE table1(var01 UNIQUE);

ALTER TABLE table2
ADD CONSTRAINT unq_names UNIQUE(names);
```

## Keys and Superkeys

Each row is has a unique combination of column values. In this sense, each row is a superkey.

Primary keys need to be columns that have no duplicates or null values. You can specify them with `PRIMARY KEY`.

```sql
CREATE TABLE artists(
  bands text PRIMARY KEY,
  records text);

CREATE TABLE artists(
  bands text,
  records text,
  PRIMARY KEY(bands, records)
);

ALTER TABLE artists
ADD CONSTRAINT bands_pk PRIMARY KEY(bands);
```

Surrogate keys are just there to be an id. Use `ADD COLUMN id serial PRIMARY KEY` to create it.

```sql
ALTER TABLE data
ADD COLUMN id serial PRIMARY KEY;

ALTER TABLE data
ADD COLUMN new_col varchar(256);
UPDATE data
SET new_col = CONCAT(name, year);
ALTER TABLE data
ADD CONSTRAINT pk PRIMARY KEY(new_col);
```

## Foreign Keys

### 1:N Relationships

A foreign key points to the primary key of another table. Their data types should be identical and the foreign key must be equal to the domain of the primary key. Use `FOREIGN KEY` and `REFERENCES table1(id_var)`.

```sql
ALTER TABLE data
RENAME COLUMN bands TO bands_id
ADD CONSTRAINT bands_fk FOREIGN KEY (bands_id) REFERENCES artists(id);
```

### N:M Relationships

We should add foreign keys to every connected table. See an example below with `REFERENCES`.

```sql
CREATE TABLE df01 (
  df_id integer REFERENCES df02 (id),
  df_names char(256) REFERENCES df02(country_names),
  another_var numeric
);

ALTER TABLE new_df
ADD COLUMN new_id integer REFERENCES df01(df_id);
```

Note that _no primary key is defined here_, as each observations can appear several times in another table.

## Referential Integrity

Referential integrity means that an id columns in table A must point to an
existing id column in table B. Referential integrity only exists when you have
two tables and is enforced with foreign keys. You can check them using
`informainformation_schema` if it's available.

```sql
SELECT constraint_name, table_name, constraint_type
FROM information_schema.table_constraints
WHERE constraint_type = 'FOREIGN KEY';
```

Note: altering a key constraint doesn't work with `ALTER COLUMN`. You need to delete the original constraint first, then create a new one.

And we've reached the end of this course!
