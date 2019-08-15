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
- [5. Update Data in a Table with SQL `UPDATE`](#5-update-data-in-a-table-with-sql-update)
  - [Looping commands](#looping-commands)
  - [Updating multiple columns](#updating-multiple-columns)
- [6. Removing Data with SQL `DELETE`, `TRUNCATE`, and `DROP`](#6-removing-data-with-sql-delete-truncate-and-drop)
  - [Deleting specific rows](#deleting-specific-rows)
  - [Deleting all rows](#deleting-all-rows)
  - [Remove the table from the database](#remove-the-table-from-the-database)
- [7. Keep Data Integrity with Constraints](#7-keep-data-integrity-with-constraints)
  - [Null values](#null-values)
  - [`UNIQUE` constraint](#unique-constraint)
  - [`FOREIGN` keys](#foreign-keys)
  - [Adding constraints to already created tables](#adding-constraints-to-already-created-tables)
  - [Viewing constraints](#viewing-constraints)
- [8. Organize Table Data with Indexes](#8-organize-table-data-with-indexes)
  - [Creating an index](#creating-an-index)
  - [Remove an index](#remove-an-index)
  - [When not to use indexes](#when-not-to-use-indexes)

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

```sql
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

```sql
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

## 6. Removing Data with SQL `DELETE`, `TRUNCATE`, and `DROP`

### Deleting specific rows

When removing rows from a table we use the `DELETE` command. As with `SELECT`
and `UPDATE` it's a looping command, so if we don't provide a condition, we'll
delete all rows in a table:

```sql
-- delete everything in the Users table
DELETE FROM Users;

-- delete only the rows that meet a condition
DELETE FROM Users WHERE first_name = 'Joe';
```

We can combine conditions using `OR` or `AND`:

```sql
-- conjunction
DELETE FROM Users
WHERE
  first_name = 'Joe' AND
  last_name = 'Soap';

-- disjunction
DELETE FROM Users
WHERE
  first_nane = 'Joe' OR
  first_name = 'John';
```

### Deleting all rows

We could use `DELETE FROM Users` to delete all the rows of the `Users` table,
but a more efficient way to remove all rows is by using `TRUNCATE`:

```sql
TRUNCATE Users;
```

`TRUNCATE` is more efficient as it doesn't iterate over each row; it simply
removes all of them.

`TRUNCATE` can also be used to remove rows from multiple tables by separating
them with commas:

```sql
TRUNCATE Table1, Table2, ... ;
```

### Remove the table from the database

`DROP` will remove a table completely from the database:

```sql
DROP TABLE Users;
```

A dropped table will not be able to be queried.

## 7. Keep Data Integrity with Constraints

Constraints are a mechanism for connecting data between tables, help with
maintaining integrity of data, and as a database scales, help to do so
efficiently.

Let's recreate our dropped table:

```sql
CREATE TABLE USERS (
  create_date date,
  user_handle uuid,
  last_name text,
  first_name text,
  -- set user_handle as the primary key for the Users table
  CONSTRAINT PK_users PRIMARY KEY (user_handle)
  --  [1]      [2]          [3]        [4]
);

-- [1] CONSTRAINT keyword which allows one to name a constraint. If a name is
       not given, most databases will generate one
-- [2] the name of the constraint. We prefix it with PK_ to indicate that it's a
       primary key
-- [3] the type of constraint we are defining - in this case, a primary key
-- [4] a list of columns used to define the primary key
```

A primary key is a column or a group of columns used to identify a row uniquely
in a table. By setting a column as a `PRIMARY KEY`, it is automatically assigned
to be `NOT NULL` and `UNIQUE` too (on Postgres, at least); rows inserted into this
table must have the specified columns populated in order for the query to succeed.

There are two ways to configure primary keys in tables:

1. if only a single column will be used as a primary key, then the column can be
   set in line

    ```sql
    CREATE TABLE MyTable (
      -- id now has NOT NULL and UNIQUE constraints
      id uiud PRIMARY KEY,
      col_1 text,
      -- ...
    );
    ```

    This is called a _column constraint_.

2. by using the `CONSTRAINT` keyword:

    ```sql
    CREATE TABLE MyTable (
      id uuid,
      col_2 text,
      -- id now has NOT NULL and UNIQUE constraints
      CONSTRAINT PK_id PRIMARY KEY (id)
    );
    ```

    This is called a _table constraint_.

    Multiple values can be used to create a primary key constraint:

    ```sql
    CREATE TABLE MyTable (
      col_1 uuid,
      col_2 text,
      col_3 date,
      -- col_1 and col_2 now have NOT NULL and UNIQUE constraints
      CONSTRAINT PK_primary_key_name PRIMARY KEY (col_1, col2)
    );
    ```

Because we now have a constraint on the `user_handle` column, if we attempted
inserting two records with the same `user_handle` then we'd get an error:

```sql
INSERT INTO Users
  (create_date, user_handle, last_name, first_name)
  VALUES
    (NOW(), 'b8a62e4e-5eb5-4fe7-a346-b768acab4a6d', 'Soap', 'Joe'),
    (NOW(), 'b8a62e4e-5eb5-4fe7-a346-b768acab4a6d', 'Doe', 'John');
ERROR:  duplicate key value violates unique constraint "pk_users"
DETAIL:  Key (user_handle)=(b8a62e4e-5eb5-4fe7-a346-b768acab4a6d) already exists.
```

### Null values

Null values can't be inserted into primary keys:

```sql
INSERT INTO Users
  (create_date, user_handle, last_name, first_name)
  VALUES
    (NOW(), null, 'Soap', 'Joe');
ERROR:  null value in column "user_handle" violates not-null constraint
DETAIL:  Failing row contains (2019-08-14, null, Soap, Joe).
```

The `NOT NULL` constraint can be used for fields where a value must be provided,
such as when a user signs up, and you require an email address:

Let's drop our `Users` table and recreate it, adding a constraint to
`first_name` and `last_name` to ensure values are provided when a record is
inserted:

```sql
DROP TABLE Users;

CREATE TABLE Users (
  create_date date,
  user_handle uuid,
  first_name text NOT NULL,
  last_name text NOT NULL,
  CONSTRAINT PK_users PRIMARY KEY (user_handle)
);
```

If we attempt to insert a record now where `first_name` or `last_name` are not
provided, we'll get an error:

```sql
INSERT INTO Users
  (create_date, user_handle, first_name, last_name)
  VALUES
    (NOW(), 'b8a62e4e-5eb5-4fe7-a346-b768acab4a6d', null, 'Soap');
ERROR:  null value in column "first_name" violates not-null constraint
DETAIL:  Failing row contains (2019-08-14, b8a62e4e-5eb5-4fe7-a346-b768acab4a6d, null, Soap).
```

Tyler recommends avoiding dealing with nulls in databases as much as possible,
as they cause a lot of trouble.

### `UNIQUE` constraint

The `UNIQUE` constraint ensures that only a single record can hold any specific
value within a table:

```sql
DROP TABLE Users;

CREATE TABLE Users (
  create_date date,
  user_handle uuid NOT NULL UNIQUE,
  first_name text NOT NULL,
  last_name text NOT NULL
);

INSERT INTO Users
  (create_date, user_handle, first_name, last_name)
  VALUES
    (NOW(), 'b8a62e4e-5eb5-4fe7-a346-b768acab4a6d', 'Joe', 'Soap'),
    (NOW(), 'b8a62e4e-5eb5-4fe7-a346-b768acab4a6d', 'Joe', 'Soap');

ERROR:  duplicate key value violates unique constraint "users_user_handle_key"
DETAIL:  Key (user_handle)=(b8a62e4e-5eb5-4fe7-a346-b768acab4a6d) already exists.
```

Let's create our table in the desired state:

```sql
DROP TABLE Users;

CREATE TABLE Users (
  create_date date NOT NULL,
  user_handle uuid PRIMARY KEY,
  last_name text,
  first_name text NOT NULL
);
```

### `FOREIGN` keys

We want to track when users make a purchase. A user who is not in our database
should never be able to make a purchase.

The following table would be incorrect, as any `uuid` could be supplied for a
user:

```
CREATE TABLE Purchases (
  create_date date NOT NULL,
  user_handle uuid,
  sku uuid NOT NULL,
  quantity int NOT NULL
);
```

To address this, one should add a foreign key constraint to a table:

```sql
CREATE TABLE Purchases (
  create_date date NOT NULL,
  -- create a column that has a foreign key constraint specifying that the value
  -- must exist in the user_handle column of the Users table
  user_handle uuid REFERENCES Users (user_handle),
  sku uuid NOT NULL,
  quantity int NOT NULL
);
```

If we attempt to insert a purchase with a `uuid` that isn't in the `Users` table
we get an error:

```sql
INSERT INTO Users
  (create_date, user_handle, first_name)
  VALUES
    (NOW(), 'b8a62e4e-5eb5-4fe7-a346-b768acab4a6d', 'Joe');

-- insert a purchase for a non-existent user
INSERT INTO Purchases
  (create_date, user_handle, sku, quantity)
  VALUES
    (NOW(), 'd7943f4b-2fda-44fd-94cf-6d72c37d6da7', '898162f1-8b6e-40e4-bd26-e264fa4ac01a', 1);
ERROR:  insert or update on table "purchases" violates foreign key constraint "purchases_user_handle_fkey"
DETAIL:  Key (user_handle)=(d7943f4b-2fda-44fd-94cf-6d72c37d6da7) is not present in table "users".

-- insert a purchase for a valid user
INSERT INTO Purchases
  (create_date, user_handle, sku, quantity)
  VALUES
    (NOW(), 'b8a62e4e-5eb5-4fe7-a346-b768acab4a6d', '898162f1-8b6e-40e4-bd26-e264fa4ac01a', 1);
-- INSERT 0 1
```

### Adding constraints to already created tables

Using the `ALTER` keyword, one can add constraints to columns and tables that
already exist.

### Viewing constraints

To see the constraints and references in Postgres:

```sql
postgres=# \d Users;
      Table "public.users"
   Column    | Type | Modifiers
-------------+------+-----------
 create_date | date | not null
 user_handle | uuid | not null
 last_name   | text |
 first_name  | text | not null
Indexes:
    "users_pkey" PRIMARY KEY, btree (user_handle)
Referenced by:
    TABLE "purchases" CONSTRAINT "purchases_user_handle_fkey" FOREIGN KEY (user_handle) REFERENCES users(user_handle)

postgres=# \d Purchases
     Table "public.purchases"
   Column    |  Type   | Modifiers
-------------+---------+-----------
 create_date | date    | not null
 user_handle | uuid    |
 sku         | uuid    | not null
 quantity    | integer | not null
Foreign-key constraints:
    "purchases_user_handle_fkey" FOREIGN KEY (user_handle) REFERENCES users(user_handle)
```

## 8. Organize Table Data with Indexes

Like the index in a book, database indexes help organise data that may be
frequently accessed, and for which we want to find information in an optimised
manner.

An analogy is how the index in a book may contain a specific word that is
important to the reader, and it then lists the pages on which that word is found
or defined.

With this thinking, its important for the author / editor of the book to
determine what is actually important to the reader so that those words are added
to the index. Too many words that aren't important will slow the reader down in
their search.

A database index is similar - the developer needs to determine which values are
important to index.

### Creating an index

Let's say that we query user data primarily by `user_handle`. `user_handle` may
be valuable to create an index from so that we can improve performance on
queries:

```sql
CREATE INDEX users_user_handle_index ON Users (user_handle);
--   [1]      [2]     [3]       [4]      [5]       [6]

-- [1] keywords to create an index
-- [2] the name of the index, which is a good idea to name something that will
       make sense. We start with the table to which the index belongs
-- [3] followed by the column from which the index is derived
-- [4] and append `_index` so that it's obvious to anyone that they are working
       with an index
-- [5] specify the table the index is for
-- [6] specify the column or columns the index is built from
```

We can see the indexes on tables using Postgres's `\d` command:

```sql
postgres=# \d Users;
      Table "public.users"
   Column    | Type | Modifiers
-------------+------+-----------
 create_date | date | not null
 user_handle | uuid | not null
 last_name   | text |
 first_name  | text | not null
Indexes:
    "users_pkey" PRIMARY KEY, btree (user_handle)
    "users_user_handle_index" btree (user_handle)
Referenced by:
    TABLE "purchases" CONSTRAINT "purchases_user_handle_fkey" FOREIGN KEY (user_handle) REFERENCES users(user_handle)
```

Indexes that use multiple columns work only for queries that use the `AND`
keyword and both columns are used in the conjunction.

### Remove an index

We use the `DROP` keyword to remove indexes:

```sql
DROP INDEX users_user_handle_index;

\d Users;
      Table "public.users"
   Column    | Type | Modifiers
-------------+------+-----------
 create_date | date | not null
 user_handle | uuid | not null
 last_name   | text |
 first_name  | text | not null
Indexes:
    "users_pkey" PRIMARY KEY, btree (user_handle)
Referenced by:
    TABLE "purchases" CONSTRAINT "purchases_user_handle_fkey" FOREIGN KEY (user_handle) REFERENCES users(user_handle)
```

### When not to use indexes

Indexes enhance performance, but they can also hurt performance. With Postgres,
indexes can be built in parallel to looksups, i.e. `SELECT` queries, but are run
prior to `UPDATE`, `INSERT`, and `DELETE` queries, which can decrease
performance.

It's best to avoid indexes for tables that:

- are small
- have frequent or large batch updates
- have columns that are frequently updated
