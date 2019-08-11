# SQL Fundamentals

Notes and annotations from Egghead's SQL Fundamentals course: [https://egghead.io/courses/sql-fundamentals](https://egghead.io/courses/sql-fundamentals)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [2. Create a Table with SQL `CREATE`](#2-create-a-table-with-sql-create)
- [3. Add Data to a Table with SQL `INSERT`](#3-add-data-to-a-table-with-sql-insert)
- [4. Query Data with the `SELECT` Command in SQL](#4-query-data-with-the-select-command-in-sql)

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
  VALUES ('2018-08-06', 'bed706b0-9eee-48d1-a67c-236046bb1cb6', 'Soap', 'Jooe');
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

   New sets of data are comma-separated.

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
