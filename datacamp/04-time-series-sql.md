# Time Series in SQL

## Building a Date

The function `GETDATE()` and `GETUTCDATE()` will return dates in local and UTC
times, respectively.

```sql 
SELECT GETDATE() date_time, GETUTCDATE() date_time_utc 
FROM dates
```

Similarly, `SYSDATETIME()` and `SYSUTCDATETIME()` returns the current local and
UTC times.

We can get specific times with built-in functions such as `YEAR`, `MONTH`, or
`DAY`. Ideally, we should use `DATEPART(YEAR/MONTH/DAY/etc, var)` as it is more
flexible. By contrast, `DATENAME` returns a string value, which is important if
we want to know the name of a particular month or day of the week. See a list
of arguments
[here](https://docs.microsoft.com/en-us/sql/t-sql/functions/datepart-transact-sql?view=sql-server-ver15).

```sql 
SELECT DATEPART(DAYOFYEAR, var) AS year
SELECT DATENAME(WEEKDAY, var) AS week_day
FROM dates
```

We can also add and subtract dates with `DATEADD`. 

```sql 
DECLARE @sometime DATETIME2(7) = '2000-02-04 09:24:32.1234567' -- 7 is the default 
SELECT DATEADD(DAY, -1, DATEADD(HOUR, -4, @sometime)) AS minus_one_day_and_four_hours
```

`DATEDIFF` calculates the difference between two dates.

```sql
DECLARE @starttime DATETIME2(7) = '2020-02-26 13:32:33'
        @endtime DATETIME2(7) = '2020-02-26 14:43:12'
SELECT DATEDIFF(MINUTE, starttime, endtime) AS minutes_passed
```

You can also round dates.

```sql 
DECLARE @SomeTime DATETIME2(7) = '2018-06-14 16:29:36.2248991';
SELECT
  DATEADD(HOUR, DATEDIFF(HOUR, 0, @SomeTime), 0) AS RoundedToHour,
  DATEADD(MINUTE, DATEDIFF(MINUTE, 0, @SomeTime), 0) AS RoundedToMinute;
```

## Calendar Tables

Calendar tables stores information for easy retrieval.

## Creating Dates


