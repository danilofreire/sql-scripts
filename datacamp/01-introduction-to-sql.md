# Introduction to SQL

## Selecting Columns

The simplest `SELECT` statement selects one column from one table:

```sql
SELECT description
FROM grid;
```

Note that you need `;` to end the statement.

You can select multiple rows by adding a `,` to the `SELECT` line.

```sql
SELECT artist_id, artist_name
FROM artists;
```

We can also specify the number of rows to return with `TOP()`, `LIMIT`, or `TOP() PERCENT`.

```sql
SELECT TOP(5) artist_id
FROM artists;

SELECT artist_id
FROM artists
LIMIT 10;

SELECT TOP(10) PERCENT artist_id
FROM artists
```

You can return unique combinations of values with `DISTINCT`.

```sql
SELECT DISTINCT artist_id
FROM artists;
```

To return all columns in a table, use `SELECT *`.

```sql
SELECT *
FROM artists;
```

Aliasing columns with `AS`.

```sql
SELECT artist_id AS musicians
FROM artists;
```

## Ordering and Filtering

We can order/sort the results with `ORDER BY` and `DESC`.

```sql
SELECT TOP(10) product, year
FROM data
ORDER BY year DESC, product;
```

We use the `WHERE` clause to ensure that the query meets our criteria.

```sql
SELECT TOP(10) product, year
FROM data
WHERE year > 1985
AND (country = 'Spain' OR country = 'Portugal')
ORDER BY year DESC, product;
```

Use `BETWEEN AND` to select values in a range.

```sql
SELECT TOP(10) product, year
FROM data
WHERE year BETWEEN 1982 AND 1990
ORDER BY year, product DESC;
```

If you use `NOT BETWEEN`, the values will be excluded from the results.

```sql
SELECT TOP(10) product, year
FROM data
WHERE year NOT BETWEEN 1982 AND 1990
ORDER BY year, product DESC;
```

Select missing cases with `IS NULL` and remove them with `IS NOT NULL`.

```sql
SELECT product
FROM data
WHERE country IS NOT NULL;
```

Use `()` to specify conditional queries.

```sql
SELECT artists, year
FROM dataset
WHERE
  (
    artists = 'AC/DC'
    AND year = 1994
  )
OR (
    artists = 'AC/DC'
    AND year >= 2000
  );
```

We use `IN` to select specific cases and `NOT IN` to exclude them.

```sql
SELECT songs, artists
FROM music
WHERE artists IN ('AC/DC', 'Metallica', 'Stratovarius'),
      release_year NOT IN (1986, 1990, 1994)
ORDER BY title;
```

To filter queries beginning with a certain character, use `WHERE LIKE 'z%'`.

```sql
SELECT songs, artists
FROM music
WHERE songs LIKE 'a%'
ORDER BY title;
```

## Aggregating Data

Calculate the column sum with `SUM() AS`.

```sql
SELECT SUM(profits) AS total_profits
FROM df;
```

You can use `COUNT() AS` to count cases. Use `COUNT(DISTINCT)` to obtain
unique values.

```sql
SELECT COUNT(DISTINCT people) AS unique_people
FROM df;
```

Use `MIN()` and `MAX()` to obtain minimum and maximum values, and `AVG()` to return the average value of the column.

```sql
SELECT AVG(profits) AS average_profits
FROM df;
```

## Strings

`LEN() AS` calculates the length of each string.

```sql
SELECT titles
  LEN(titles) AS titles_length
FROM df;
```

`LEFT(var, n) AS` gets the first letters in the string. `RIGHT(var, n)` does the same from the right.

```sql
SELECT titles
  LEFT(titles, 20) AS titles_left
FROM df;
```

`CHARINDEX('char', var)` finds a specific character in a string.

```sql
SELECT titles
  CHARINDEX('p', titles) AS titles_with_p
FROM df;
```

`SUBSTRING(var, nchars_start, nchars_end)` searches for characters in cases.

```sql
SELECT titles
  SUBSTRING(titles, 5, 10) AS titles_chars
FROM df;
```

Lastly, `REPLACE(var, 'old', 'new')` replaces characters.

```sql
SELECT titles
  REPLACE(titles, 's', 'z') AS title_with_z
FROM df;
```

## Grouping and Having

`GROUP BY`, as you imagine, groups results by a given variable.

```sql
SELECT country, firms, year, SUM(quarter_profits) AS year_profits
FROM df
WHERE firms LIKE '%Bank%'
AND quarter_profits IS NOT NULL
GROUP BY country, year;
```

`HAVING` is used after `GROUP BY` to restrict the queries after the results. Remember: `WHERE` appears _before_ `GROUP BY` and `HAVING` _after_ it.

```sql
SELECT country, firms, year, SUM(quarter_profits) AS year_profits
FROM df
WHERE firms LIKE '%Bank%'
AND quarter_profits IS NOT NULL
GROUP BY country, year;
HAVING SUM(year_profits) > 100000
```

## Joining Tables

The first type of join is `INNER JOIN ON`. The same syntax works for `LEFT JOIN` and `RIGHT JOIN`.

```sql
SELECT table1.column1, table1.column2, table2.column1
FROM df
INNER JOIN table2 ON table2.column1 = table1.column1;
```

Remember that:

* `INNER JOIN` will only return matching rows
* `LEFT JOIN` and `RIGHT JOIN` will include all rows of the main table and add `NULL` if no matching is found.

`UNION` and `UNION ALL` combines different tables. The first remove duplicates.

```sql
SELECT country_id AS ID, hdi AS HDI, gdp AS GDP, 'countries' AS SOURCE_TABLE
FROM countries
UNION
SELECT nation_id AS ID, hdindex AS HDI, GDP, 'nations' AS SOURCE_TABLE
FROM nations;
```

## Creating Tables

Use `CREATE TABLE` and a unique name, column name, type, and size.

```sql
CREATE TABLE new_df(birth_year DATE, name VARCHAR(20), age INT, male BIT);
```

How to populate a table? First, we can use the `INSERT INTO` command along with `VALUES`.

```sql
CREATE TABLE new_df(birth_year DATE, name VARCHAR(20), age INT, male BIT)
INSERT INTO new_df (birth_year, name, age, male)
VALUES (1990, 'John', 20, 1);
```

We can also use `INSERT SELECT` to add columns from another table.

```sql
CREATE TABLE df02 (var1, var2)
SELECT column1, column2
FROM df01
WHERE column1 > 1000, column2 = 'male';
```

`UPDATE` is an important command too. Use it with `SET` and `WHERE`.

```sql
CREATE TABLE df02 (var1, var2)
UPDATE df02
SET var1 = 10,
WHERE var1 = 0;
```

Finally, `DELETE FROM`. Note: there is no confirmation prompt! It's a good idea to run `SELECT` again to make sure the value was deleted.

```sql
DELETE FROM df02
WHERE var1 = 99
SELECT var1
FROM df02;
```

## Temporary Tables

Preface the table name with `#` and delete it with `DROP TABLE`.

```sql
SELECT var1 INTO #temporary_table
FROM df01
WHERE var1 <= 5000;
```

We can also use `DECLARE` to speed up the querying process. Create a variable with `@`, then use `SET` to add your search term. Update the `SET` value whenever you need it.

```sql
DECLARE @band VARCHAR(20)
SET @band = 'AC/DC' -- just change the name and run the code again
SELECT artist, track, record, year
FROM rock
WHERE artist = @band
```

A more complex example:

```sql
DECLARE @beginning DATE
DECLARE @end DATE
DECLARE @sick INT;
SET @beginning = '2019-06-22'
SET @end  = '2020-12-02'
SET @sick =  1000;

SELECT patients, country, region, period
FROM epidemics
WHERE period BETWEEN @beginning AND @end
AND patients >= @sick;
```

...And that's all! Phew, that was a long lesson, but I've learned quite a lot!
