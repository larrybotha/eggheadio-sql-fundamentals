# SQL Fundamentals

Notes and annotations from Egghead's SQL Fundamentals course: [https://egghead.io/courses/sql-fundamentals](https://egghead.io/courses/sql-fundamentals)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [2. Create a Table with SQL `CREATE`](#2-create-a-table-with-sql-create)
- [3. Add Data to a Table with SQL `INSERT`](#3-add-data-to-a-table-with-sql-insert)
- [4. Query Data with the `SELECT` Command in SQL](#4-query-data-with-the-select-command-in-sql)
  - [Query specific columns from a table](#query-specific-columns-from-a-table)
  - [Querying with built-in functions](#querying-with-built-in-functions)
  - [Aliasing columns in a query](#aliasing-columns-in-a-query)
  - [Query number of rows in the table](#query-number-of-rows-in-the-table)
  - [Filter by values](#filter-by-values)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 2. Create a Table with SQL `CREATE`

```bash
postgres=# CREATE TABLE Users (
  create_date date,
  user_handle uuid,
  last_name text,
  first_name text
);
```

To view tables in the current database

```bash
-- view all tables in the current
postgres=# \d

-- view a specific table
postgres=# \d [TableName]
```

Data types to evaluate:

- [Postgres datatypes](https://www.postgresql.org/docs/9.5/datatype.html)
- [MySQL datatypes](https://dev.mysql.com/doc/refman/8.0/en/data-types.html)

## 3. Add Data to a Table with SQL `INSERT`

```bash
postgres=# INSERT INTO Users
  (create_date, user_handle, last_name, first_name)
  VALUES ('2018-08-06', 'bed706b0-9eee-48d1-a67c-236046bb1cb6', 'Soap', 'Joe');
         ('2018-08-06', '03e95b02-009c-4568-9c23-17e4e96f8e3f', 'John', 'Red');
```

1. begin the insert statement with:

    ```sql
    INSERT INTO [TableName]
    ```

2. provide the names of the columns into which you're going to insert values:

    ```sql
    (col_1, col_2, col_x, col_y)
    ```

    This is not required, but is a best practice so that one knows where values
    are being inserted.

    Furthermore, not all values need to be provided, and values can be provided to
    specific columns. By naming the specific columns that values are going to be
    inserted into, we can specify any order for the columns.

    Without specifying the columns, we need to insert values in the order in which
    columns are defined for the table.

3. provide the values for the specified, preceding the values the `VALUES`
   keyword:

     ```sql
     VALUES (100, 'Sam', 'porpoise'),
            (2, 'John', 'turtle');
     ```

    If values are inserted that don't match the data types the columns are
    defined with, we'll get an error.

    Muliple sets of data are inserted by separating each set with commas.

We don't need to manually insert values, either; we can use built-in functions
to generate values instead.

e.g. both MySQL and Postgres provide a `NOW()` function to generate a date:

```sql
INSERT INTO Person
  (first_name, created_at)
  VALUES
    ('Jacob', NOW());
```

`NOW()` returns a `DATETIME` type with the current time.

Bulk inserting of data is possible, too, such as from CSV data. With Postgres
the `COPY` command can be used to assist with this.

## 4. Query Data with the `SELECT` Command in SQL

The quickest way to pull data out of a table is by using:

```sql
SELECT * FROM [TableName];
--    [1]         [2]

-- [1] - which columns to select; * says "get all columns"
-- [2] - which tables to select the columns from
```

e.g.

```sql
SELECT * FROM Users;

 create_date |             user_handle              | last_name | first_name
-------------+--------------------------------------+-----------+------------
 2018-08-06  | bed706b0-9eee-48d1-a67c-236046bb1cb6 | Soap      | Joe
 2018-08-06  | 8b9cfc4f-4844-47d1-9cb1-8e730034e5f8 | John      | Red
(2 rows)
```

### Query specific columns from a table

To query specific columns, provide the column names:

```sql
SELECT first_name, last_name FROM Users;

 first_name | last_name
------------+-----------
 Joe        | Soap
 Red        | John
(2 rows)
```

If a column that doesn't exist is queried, we get an error:


```sql
SELECT first_name, last_name, middle_name from Users;
ERROR:  column "middle_name" does not exist
LINE 1: SELECT first_name, last_name, middle_name from Users;
                                      ^
```

### Querying with built-in functions

Many SQL databases have built-int functions that can be returned in a query,
despite them not being columns in the queried table:


```sql
SELECT first_name, last_name, current_time FROM Users;

 first_name | last_name |       timetz
------------+-----------+--------------------
 Joe        | Soap      | 21:39:34.386055+00
 Red        | John      | 21:39:34.386055+00
(2 rows)
```

### Aliasing columns in a query

Instead of returning a query with the names of the columns as-is, we can specify
an alias for columns for whatever reason we choose:

```sql
SELECT first_name as f, last_name as l FROM Users;

  f   |  l
------+------
 Joe  | Soap
 Red  | John
(2 rows)
```

### Query number of rows in the table

Use the `COUNT` function to retrieve the number of rows in a table:

```sql
SELECT COUNT(*) FROM Users;

 count
-------
     2
(1 row)

-- or, count by columns in the table that have values:

SELECT COUNT(first_name) FROM Users;

 count
-------
     2
(1 row)
```

### Filter by values

First, let's insert some data with columns that match our existing data:

````sql
INSERT INTO Users
  (first_name, last_name, user_handle, create_date)
  VALUES
  ('Jane', 'Soap', '0aecc6ea-d182-4d5a-af06-6c7b34d5dc24', NOW())
```

We can now get distinct values for a specific column by using the `DISTINCT`
function:


```sql
SELECT DISTINCT(last_name) FROM Users;

 last_name
-----------
 Soap
 John
(2 rows)
```

We can combine query functions, too. Let's get the number of distinct names and
surnames:

```sql
-- # distinct names
SELECT COUNT(DISTINCT(first_name)) as num_distinct_names FROM Users;
 num_distinct_names
--------------------
                  3
(1 row)

-- # distinct surnames
SELECT COUNT(DISTINCT(last_name)) as num_distinct_last_names FROM Users;
 num_distinct_last_names
-------------------------
                       2
(1 row)
```

## 5. Update Data in a Table with SQL `UPDATE`

Let's insert a user that happens to have a `user_handle` column that's the
same as another:

```sql
INSERT INTO Users
  (create_date, user_handle, last_name, first_name)
  VALUES
    (NOW(), 'bed706b0-9eee-48d1-a67c-236046bb1cb6', 'Mike', 'Turner');
```

We want `user_handle` to be unique for all users, so we need to update the
user's handle so that it's distinct from our existing user.

Postgres has a handy extension that we can make use of to automatically generate
a uuid for us:

``sql
CREATE EXTENSION "uuid-ossp";
CREATE EXTENSION
```

With this extension created, we can update the `Users` table:

```sql
UPDATE Users SET user_handle = uuid_generate_v4();
--      [1]  [2]     [3]              [4]
UPDATE 4

-- [1] the table of the column we want to update
-- [2] the SET keyword precedes the column name we want to update
-- [3] the actual column that we want updated
-- [4] the value we want to update the column to
```

As per the output, we can see that 4 rows were updated. This could lead to
problems, as we may be destroying or changing data that shouldn't be changed. We
should be using the `WHERE` clause to ensure we're changing only the column
value of the specific row that should be changed.

### Looping commands

The following commands are looping commands:

- `SELECT`
- `UPDATE`
- `DELETE`

These commands will loop over all matching rows, so if we aren't careful, and we
don't filter rows by only those we explicitly want affected by these commands,
we could end up changing or removing critical data.

For the above command we _should_ have instead used:

```sql
UPDATE Users SET user_handle = uuid_generate_v4() WHERE last_name = 'Turner';
```

Unless a condition is provided, a looping command will apply to every row.

### Updating multiple columns

`SET` can be passed multiple column names and values to assign in a single
command:

```sql
UPDATE Users SET
  user_handle = uuid_generate_v4(),
  first_name = 'Danny'
  WHERE
    last_name = 'clark';
```
