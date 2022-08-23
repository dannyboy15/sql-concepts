# SQL

_The following explanations are based on Redshift SQL. While SQL is a commonly
accepted standard, there are some differences in how some engines (e.g.
Redshift) interpret certian queries or commands._

## SQL Basics

_See Definitions for help with common words_

SQL statements are case insensitive. That is, `SELECT` and `select` are
processed in the same way. Also, whitespace (i.e. _extra_ spaces, tabs, new
lines) are ignored. Use a semicolon (`;`) to terminate a statement.

Comments are lines of text that are ignored by the SQL engine. They are useful
for adding context for yourself or your teammates.

For single-line comments use two (2) dashes at the beginning of the line.

```sql
-- this is a single-line comment and will be ignored by the SQL engine
```

For multi-line comments use a forward slash followed by an asterisk to start the
comment section and use an asterisk followed by a forward slash to close out the
comment.

```sql
/* this is a multi-line comment
   and it can span many lines and is still ignored */
```

### SELECT

The most useful and common command for exploring or analyzing data is the
`SELECT` command.

This command can be as simple as the following queries. Sometimes, these are
useful for confirming you are successfully connected to the database.

```sql
SELECT 'some text';
```

| ?column?  |
| --------- |
| some text |

```sql
SELECT current_date;
```

| date       |
| ---------- |
| 2022-01-01 |

To request data from a table specify the columns (in a comma-separated list) of
data that you want and the table from which to get them. You will have to
include the name of the schema and the name of the table joined by a period
(`.`). Make sure to follow the format outlined below; the keywords are in
uppercase for clarity.

```sql
SELECT
    column_1, -- this is the field name
    column_2,
    Column_n -- note that the last column doesn’t include a comma after it
FROM
    schema_name.table_name;
```

You can also use the SQL wildcard (`*`) to return all the columns without having
to list the out.

```sql
SELECT
    * -- return all the columns
FROM
    schema_name.table_name;
```

You can add aliases to column names (as a way to rename them), or table names
(to more conveniently reference them). To do so, add the alias after the
column/table name. You can optionally add the `AS` keyword between the name and
the alias. You can use a table’s alias (or the complete name, for that matter)
to specifically reference a column. _This is more useful, and sometimes
required, when you join tables._

```sql
SELECT
    column_1 AS column_one, -- rename this column
    column_2 column_two, -- rename this column, not using AS keyword
    tbl.column_n -- reference the column using the table alias
FROM
    schema_name.table_name AS tbl;
```

### Functions

As you are selecting data, you may want to modify or transform the data. You can
use standard functions provided by the database (or create your own custom
ones).

String functions are functions that are applied to each value independently. For
example, the following function changes all the letters to uppercase (regardless
of what case they had originally).

```sql
SELECT
    UPPER(lowercase_data) -- change all letter to uppercase
FROM
    schema_name.table_name;
```

So if we had this table,

|                                |
| ------------------------------ |
| this is all lowercase          |
| This has some UPPERcase letter |
| ALL UPPERCASE                  |

and we applied the `UPPER` function, the resulting data would look like

|                                |
| ------------------------------ |
| THIS IS ALL LOWERCASE          |
| THIS HAS SOME UPPERCASE LETTER |
| ALL UPPERCASE                  |

Some common/useful string functions include (a full list can be found in the
[Redshift documentation](https://docs.aws.amazon.com/redshift/latest/dg/String_functions_header.html)):

| function                      | description                                                                                    |
| ----------------------------- | ---------------------------------------------------------------------------------------------- |
| `UPPER` / `LOWER` / `INITCAP` | Change letters to uppercase /lowercase / uppercase the first letter of each word, respectively |
| `CONCAT`                      | Concatenate the two values. (You can also use `\|\|`, e.g. `value_1 \|\| value_2`)             |
| `LENGTH`                      | Count the number of characters in a string                                                     |
| `REPLACE`                     | Replace all occurrences of a string                                                            |
| `LTRIM` / `RTRIM`             | Remove characters from the left / right, respectively                                          |
| `SPLIT_PART`                  | Split a string into parts using a delimiter                                                    |
| `SUBSTRING`                   | Get part a string                                                                              |

Aggregate functions are functions that are applied to an entire column. For
example, to get a count of the number of records in a table.

```sql
SELECT
    count(*)
FROM
    schema_name.table_name;
```

Some common/useful aggregate functions include (a full list can be found in the
[Redshift documentation](https://docs.aws.amazon.com/redshift/latest/dg/c_Aggregate_Functions.html)):

| function      | description                                         |
| ------------- | --------------------------------------------------- |
| `COUNT`       | Count the number of non-null values                 |
| `SUM`         | Sum the values in the column                        |
| `MIN` / `MAX` | Get the minimum / maximum of the values in a column |

### CASE Expression

If you want to use logic (i.e. if/then) to modify a column, you can use the
`CASE` expression to do so. It allows you to provide ‘cases’ and their
corresponding outputs. For example, suppose you have the following table.

| voter_id | age |
| -------- | --- |
| voter_1  | 16  |
| voter_2  | 25  |
| voter_3  | 44  |
| voter_4  | 17  |

You can use the following query to determine which voters are age-eligible to
vote.

```sql
SELECT
    voter_id,
    CASE
        WHEN age >= 18 then 'yes'
        WHEN age < 17 then 'no'
    END AS is_eligible_voter
FROM
    voter_schema.voter_table;
```

You can use a `CASE` expression wherever you are using a column, that is, in the
`WHERE` clause or in a `JOIN`.

### Joining Tables

At times you will want to get data from multiple tables. To accomplish this you
will have to join the tables using a common column, often times and ID column.
There are multiple types of joins, including inner join, outer join, left/right
joins. See Figure 2 for more details on joins.

```sql
SELECT
    column_from_first_table,
    column_from_second_table
FROM
    schema_name.table_name AS left_tbl
JOIN
    another_schema.another_table AS right_tbl
ON left_tbl.id = right_tbl.id_column;
```

If the column that is being used to join the tables has the same name, you can
instead use the `USING` clause instead.

```sql
SELECT
    column_from_first_table,
    column_from_second_table
FROM
    schema_name.table_name AS left_tbl
JOIN
    another_schema.another_table AS right_tbl
USING(id); -- note the difference here
```

Note, when a column with the same name exists in both tables, you will
explicitly have to add from which table you want the column.

```sql
SELECT
    l.column_one, -- use alias to indicate it comes from 'l' table
    r.column_one
FROM
    schema_name.table_name AS l
JOIN
    another_schema.another_table AS r
USING(id);
```

### Filtering

Sometimes you may want to get a subset of the rows. For example, you may only
want rows in a certain timeframe. You can use the `WHERE` clause to filter down
the data that is returned.

```sql
SELECT
    *
FROM
    schema_name.table_name AS l
JOIN
    another_schema.another_table AS r
USING(id)
WHERE
    state = 'CA';
```

You can use `AND` and `OR` to combine various conditions. Keep in mind that
`AND` has precedence over `OR`. To help clarify and make the clause more
explicit, you can use parentheses to wrap your conditions.

```sql
SELECT
    *
FROM
    schema_name.table_name AS l
JOIN another_schema.another_table AS r
USING(id)
WHERE
    (county = 'Orange' OR zipcode = 123455) OR is_registered is TRUE;
```

### Aggregations

As noted earlier, aggregate functions are applied at the column level. You can
take things further and group by a column. This applies the functions
independently to each group. For example, if you wanted to get the number of
records for each day, you could do something like this,

```sql
SELECT
    date_column,
    COUNT(*)
FROM
    schema_name.table_name
GROUP BY date_column -- you can also use the column index, e.g. 1
;
```

The resulting data could look like,

| date_column | count |
| ----------- | ----- |
| 2022-01-01  | 150   |
| 2022-02-01  | 67    |
| 2022-03-01  | 183   |

### Sorting

To sort data, add the `ORDER BY` clause and specify a list of columns. You can
also specify whether you want the order to be ascending (`ASC`) or descending
(`DESC`). If not specified, the order is set to ascending.

```sql
SELECT
    state,
    date,
    amount
FROM
    schema_name.table_name

-- you can also use the column index, e.g. 1
-- e.g. 1 asc, 2 desc
ORDER BY state asc, date desc;
```

[add example of using where, group and sort]

### CREATE

There are two (2) main ways to create a table (or view). The first way requires
specifying the columns and their types to create an empty table\*. The second
way requires using a query the is used to create the table and populate it.

\* Note: if you create an empty table and want to add data to it, you will need
to run a COPY command. This is more advanced.

### Creating an Empty Table

To create a table you will need the schema\* and table name for the new table,
as well as the columns and their types. [link to full list of types]

[Conversion and coercion]

Column type

| type        | description                                                                                                                                               |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CHAR`      | A fixed-length string. Specific the length in parenthesis, e.g. `CHAR(2)`.                                                                                |
| `VARCHAR`   | A variable-length string value. Specify the max length in parenthesis, e.g. `VARCHAR(50)`. If none is provided, the max allowed value of `65535` is used. |
| `BOOL`      | A logical `TRUE` or `FALSE` values                                                                                                                        |
| `INT`       | An integer                                                                                                                                                |
| `FLOAT`     | A floating point value (commonly known as decimal numbers). For greater precision use `DECIMAL`.                                                          |
| `DATE`      | A date value represented at `'2022-01-01'`, i.e. 4-digit year, 2-digit month, 2-digit day.                                                                |
| `TIMESTAMP` | A date and time value represented as `'2022-01-01 00:00:00'`, i.e. a date value with 24-hour format for the hour, then minutes and seconds                |

\* If you do not provide a schema name, the table will be created in the
`public` schema.

```sql
CREATE TABLE my_schema.my_new_table (
    first_name varchar(100),
    last_name varchar(100),
    age int,
    date_of_birth date
);
```

### Creating a Table From a Query

You can create a table from existing data. This will create and populate a table
using the specified query. When using this command you do not have to specify
the column types, they are inherited from the source table\*. You can change the
column names (in the new table) by using an alias. For example, this statement
would create a table called `my_schema.my_new_table_2` using data from
`my_schema.my_new_table` and would rename `first_name` to `firstname` in the
newly created table.

```sql
CREATE TABLE my_schema.my_new_table_2 AS (
    SELECT
        first_name as firstname, -- you can rename the columns using aliases
        last_name,
        age,
        date_of_birth
    FROM
      my_schema.my_new_table
);
```

[add statement for finding duplicates]

## Advanced SQL

- Common Table Expressions (CTE)
- Window functions

## Definitions

- **SQL**: Structured Query Language, commonly pronounced S-Q-L or Seequel
- **statement**: a generic instruction to a database, i.e. select, create
- **query**: a statement that asks the database to return some data
- **schema**: a logical structure in a database for grouping tables/views
  together.
- **table**: a database object that is made up of rows and columns
- **view**: a virtual table that is defined by a query
- **null**: a non-existent value, different from an empty string (`''`)

## Appendix

- [SQL Joins](https://qph.fs.quoracdn.net/main-qimg-4b2d1255236a6511e1112263e398034b)
