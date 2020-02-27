# Manipulating Data

## Data Types

There are many data types you can use. You can define:

  - Exact numerics
  - Approximate numerics
  - Date and time
  - Characters
  - Unicode strings
  - Binary data
  - Other types

Exact types can have several types. _Precision_ means the number of decimal
digits that will be stored, while _scale_ is the number of decimal digits after
the decimal. The more precision you specify, the more data you will use.

ASCII stores strings in English characters, and Unicode stores characters in
many languages, such as Chinese or Arabic. Unicode data types start with `n`,
such as `nchar`, `nvarchar`, `ntext`.

The `date` data type stores dates, `time` stores time (such as hours or
minutes), and `datetime2` stores dates and times together.

To compare two variables, we need them to have the same data type.
