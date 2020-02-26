# Intermediate SQL Server

## Summarising Data

As we have seen in the first course of this series, SQL has many commands to summarise data. For instance, we have `MIN` to get the minimum values of a column, `MAX` to retrieve the maximum, and `AVG` to return the average. Remember that you always need to create another variable with the results.

To subset your data, use `WHERE` and pass a condition. For instance, if you want to know the average age of the Greek population, type:

```sql
SELECT AVG(age) AS avg_age
FROM indicators
WHERE country = 'Greece'
```

Note that you don't need to select the variable in the `WHERE` condition.
If you want to group results by country then filter them later, use `HAVING` instead of `WHERE`. Note that here you need to select the column you will group the cases by.

```sql
SELECT MIN(age) AS min_age, MAX(age) AS max_age, AVG(age) AS avg_age, country
FROM indicators
GROUP BY country
HAVING AVG(age) > 35
```

## Missing Data

Missing data is represented by `NULL`. To delete missing values, add `WHERE var IS NOT NULL` to the code.

Interestingly, a blank value _is not_ a null value. Blank values may show up in rows containing text and might contain information.

You can replace null values in two ways, with `ISNULL` or `COALESCE`. The first allows you to impute anything, the latter returns the first non-missing value from other variables you select.

```sql
-- Replace all missing cases with 'Unknown'.
SELECT gdp, country
ISNULL(country, 'Unknown') AS new_country
FROM indicators;

-- Use other variables to impute missing values, SQL will search them in that order.
SELECT gdp, country
COALESCE(country, nation_id, other_id) AS new_country
FROM indicators;
```

## Case Statements

Case statements have the following syntax:

```sql
CASE WHEN this_is_true THEN result
  ELSE something
END

-- Example.
SELECT country
CASE WHEN country = 'US' or country = 'United States' THEN 'USA'
     ELSE country
     END AS imputed_country
FROM indicators;
```

We can also use `CASE WHEN` to create bins.

```sql

SELECT country, life_exp
CASE WHEN life_exp < 50 THEN 1
     WHEN life_exp >= 51 AND life_exp <= 70 THEN 2
     WHEN life_exp >= 71 AND life_exp <= 80 THEN 3
     WHEN life_exp >= 81 THEN 4
     ELSE 5
     END AS bins
FROM indicators
WHERE year = 2000;
```

## Count and Total

The `COUNT` function gives you the total number of observations in a table and
`COUNT(DISTINCT col)` returns the number of distinct values of a column.

```sql
SELECT COUNT(*) FROM indicators;
SELECT COUNT(DISTINCT country) AS countries FROM indicators;

-- Combining COUNT with GROUP BY.
SELECT COUNT(*) AS rows_by_country FROM indicators
GROUP BY country
HAVING rows_by_country > 100
ORDER BY country DESC;
```

## Maths with Dates

Two important functions: `DATEADD()`, which adds two dates and returns a date, and `DATEDIFF()`, which returns the difference between two dates in numeric format. Use `YYYY` for year, `MM` for month and `DD` for days.

```sql
DATEADD(DD, 30, '2020-02-24') -- Adds 30 days to today.
DATEADD(MM, -2, '2020-02-24') -- Subtracts 2 months from today.

SELECT DATEDIFF(YYYY, '2020-02-24', '1990-06-19') AS diffs

-- You can use the commands with variables.
SELECT OrderDate,
       DATEADD(DD, 2, ShipDate) AS DeliveryDate
FROM Shipments
```

## Rounding Numbers

This is very easy. If you are familiar with `R` you will get it quickly: just use `ROUND(var, number)`.

```sql
SELECT gdp ROUND(gdp, 0) AS gdp_rounded FROM indicators
```

## Maths Functions

Some basic mathematical operations:

* `ABS` returns the absolute value
* `SQRT` returns the square root
* `SQUARE` returns the square
* `LOG(n, base)` returns the log


## While Loops

`WHILE` conditions also work the same as in other computer languages. You need
to create a variable with `DECLARE @var data_type`. To break a loop, use `IF
... BREAK` (optional). You may find an example below.

```sql
DECLARE @age INT
SET @age = 1
WHILE @age < 30
  BEGIN
    SET @age = @age + 1
    IF @age >= 18
       BREAK
  END
SELECT @ctr;
```

## Derived Table

Derived tables are virtual tables created by using `SELECT`. They are similar
to temporary tables, but they are often easier to create. Each derived table
should have its own alias.

```sql
SELECT productName, sales
FROM
    (SELECT productCode, ROUND(SUM(quantityOrdered * priceEach)) sales
    FROM orderdetails
    INNER JOIN orders USING (orderNumber)
    WHERE YEAR(shippedDate) = 2003
    GROUP BY productCode
    ORDER BY sales DESC
    LIMIT 5) top5products2003 /* derived table name */
INNER JOIN products USING (productCode);

SELECT * FROM healthcare hc_01 /* any name */
JOIN (SELECT age, MAX(height) as max_height
      FROM healthcare GROUP BY age) hc_02
ON hc_01.height = hc_02.height
AND hc_01.age = hc_02.age;

-- Last example. See how it saves time as you only need to use CASE WHEN once.
SELECT FilmLength, COUNT(*) AS NumberOfFilms
FROM (SELECT
        CASE WHEN FilmRunTimeMinutes < 100 THEN 'Short'
             WHEN FilmRunTimeMinutes < 150 THEN 'Medium'
             WHEN FilmRunTimeMinutes < 200 THEN 'Long'
        ELSE 'Epic'
        END AS FilmLength
      FROM tblFilm) AS FilmLengths
GROUP BY FilmLength;
```

## Common Table Expressions (CTEs)

CTEs are a modern alternative to derived tables. They are temporary tables that
can simplify large queries and help with complex joins. A CTE always returns a
result set. They start with the `WITH` statement followed by `AS` and `SELECT`,
`INSERT`, `UPDATE` or `DELETE` statements.

```sql
-- Simple example
WITH cte_sales AS(
  SELECT staff_id, COUNT(*) order_count FROM sales_orders
  WHERE YEAR(order_date) = 2018
  GROUP BY staff_id
)
SELECT AVG(order_count) AS avg_order_by_staff
FROM cte_sales;

/* Using multiple CTEs in the same query */
WITH Sales_CTE (SalesPersonID, TotalSales, SalesYear) -- Create 3 vars
AS( -- Define first CTE query
      SELECT SalesPersonID, SUM(TotalDue) AS TotalSales, YEAR(OrderDate) AS SalesYear
      FROM Sales.SalesOrderHeader
      WHERE SalesPersonID IS NOT NULL
        GROUP BY SalesPersonID, YEAR(OrderDate)

),  -- Use a comma to separate multiple CTE definitions.
    -- Second CTE query, which returns sales quota data by year for each sales person.
Sales_Quota_CTE (BusinessEntityID, SalesQuota, SalesQuotaYear)
AS(
      SELECT BusinessEntityID, SUM(SalesQuota)AS SalesQuota, YEAR(QuotaDate) AS SalesQuotaYear
      FROM Sales.SalesPersonQuotaHistory
      GROUP BY BusinessEntityID, YEAR(QuotaDate)
)

    -- Define the outer query by referencing columns from both CTEs.
SELECT SalesPersonID,
       SalesYear,
       FORMAT(TotalSales,'C','en-us') AS TotalSales,
       SalesQuotaYear,
       FORMAT (SalesQuota,'C','en-us') AS SalesQuota,
       FORMAT (TotalSales -SalesQuota, 'C','en-us') AS Amt_Above_or_Below_Quota
FROM Sales_CTE
JOIN Sales_Quota_CTE ON Sales_Quota_CTE.BusinessEntityID = Sales_CTE.SalesPersonID
AND Sales_CTE.SalesYear = Sales_Quota_CTE.SalesQuotaYear
ORDER BY SalesPersonID, SalesYear;
```

## Window Functions

Windows allows uses to evaluate each data group separately. It is crated with
`OVER(PARTITION BY var) AS x`. You can use many aggregation functions, like
`SUM`, `COUNT`, or `LAG`, `LEAD`, `FIRST VALUE`, `LAST_VALUE`, `ROW_NUMBER`,
etc.

```sql
SELECT product_id, year, sales_per_quarter
 SUM(sales_per_quarter)
 OVER(PARTITION BY year ORDER BY product_id) AS sales_per_year
FROM sales_total;

-- Lag
SELECT sales_person, year, quota
  LAG(quota)
  OVER(PARTITION BY year ORDER BY sales_person) AS lagged_quota
FROM sales_total;

-- Calculate row number
SELECT sales_person, year, quota
  ROW_NUMBER()
  OVER(PARTITION BY year ORDER BY sales_person) AS lagged_quota
FROM sales_total;
```

We can also use windows functions to calculate statistical functions, such as
`STDEV`. Here is an example:

```sql
SELECT sales_person, year, quota
  STDEV(quota)
  OVER() AS sd -- no column to partition by
FROM sales_total;

-- Calculate mode, not a ready-made function
WITH sales_mode AS(
  SELECT sales_per_person, year, quota
  ROW_NUMBER()
  OVER(PARTITION BY sales_per_person ORDER BY sales_per_person) AS mode_sales_per_person
  FROM sales_total
)
SELECT sales_per_person, mode_sales_per_person AS mode
FROM sales_mode
WHERE mode_sales_per_person IN (SELECT MAX(mode_sales_per_person) FROM sales_mode);
```

Wow, that was a hard lesson. I have to go back to it a few times before I'm
sure I fully understand it. But so far, so good! SQL is not so hard after all!
