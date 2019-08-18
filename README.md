# SQL Fundamentals

Notes and annotations from Egghead's SQL Fundamentals course: [https://egghead.io/courses/sql-fundamentals](https://egghead.io/courses/sql-fundamentals)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Running a local db](#running-a-local-db)
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
- [9. Select Grouped and Aggregated Data with SQL](#9-select-grouped-and-aggregated-data-with-sql)
  - [Grouping columns with unique content](#grouping-columns-with-unique-content)
  - [Adding column names to a `GROUP BY` query to make results more descriptive](#adding-column-names-to-a-group-by-query-to-make-results-more-descriptive)
  - [Grouping by multiple columns](#grouping-by-multiple-columns)
  - [Built-in aggregate functions](#built-in-aggregate-functions)
    - [`MIN`](#min)
    - [`MAX`](#max)
  - [Aggregate functions and multiple columns](#aggregate-functions-and-multiple-columns)
- [10. Conditionally Select out Filtered Data with SQL Where](#10-conditionally-select-out-filtered-data-with-sql-where)
  - [Conjunctions and disjunctions](#conjunctions-and-disjunctions)
  - [Comparison operators](#comparison-operators)
  - [Comparison predicates](#comparison-predicates)
    - [`NULL` predicate](#null-predicate)
    - [`BETWEEN` predicate](#between-predicate)
  - [Row and array comparison](#row-and-array-comparison)
    - [`IN` / `NOT IN`](#in--not-in)
    - [`ANY` / `SOME`](#any--some)
    - [`ALL`](#all)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Running a local db

1. Install Docker
2. Run `docker-compose`
3. Connect to the `psql` process running inside the container

```bash
# start the container
$ docker-compose up

# connect to the psql instance
$ docker-compose exec sql_fundamentals psql -U postgres
```

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

## 9. Select Grouped and Aggregated Data with SQL

Let's say we have 4 users in our `Users` table:

```sql
postgres=# SELECT * FROM Users;
 create_date |             user_handle              | last_name | first_name
-------------+--------------------------------------+-----------+------------
 2019-08-17  | 649e9396-1cb0-43dd-a907-7dc6fd23ddc7 | Soap      | Jane
 2019-08-17  | c85b174b-cc34-4948-b5c8-7acdf44ceb0f | Doe       | Jane
 2019-08-17  | 1c662040-f4dd-4188-9a93-692bee9b2888 | Soap      | John
 2019-08-17  | cc7f43e0-0bb3-4df6-9cbc-19f99f2dd82c | Doe       | John
(4 rows)
```

If we wanted to count the number of users by the same `last_name`, we can
use the `GROUP BY` keyword to group rows by column:

```sql
SELECT COUNT(*) FROM Users GROUP BY last_name;

 count
-------
     2
     2
(2 rows)
```

### Grouping columns with unique content

Grouping by a column that contains unique values will return a table
with a count of 1 for every row:

```sql
SELECT COUNT(*) FROM Users GROUP BY user_handle;

 count
-------
     1
     1
     1
     1
(4 rows)
```

### Adding column names to a `GROUP BY` query to make results more descriptive

Grouping by a column alone doesn't give us very much information. If we group by
`last_name` we don't know what the counts for the values represent.

We can add a column to our `SELECT` statement to output any columns we want
returned in the query:


```sql
SELECT COUNT(*), last_name FROM Users GROUP BY last_name;

 count | last_name
-------+-----------
     2 | Soap
     2 | Doe
(2 rows)
```

If we attempt to add another column to return in the results, but that column is
not in the `GROUP BY` statement, we'll get an error:

```sql
SELECT COUNT(*), last_name, first_name FROM Users GROUP BY last_name;

ERROR:  column "users.first_name" must appear in the GROUP BY clause or be used in an aggregate function
LINE 1: SELECT COUNT(*), last_name, first_name FROM Users GROUP BY l...
```

Because more than one row can be in a single group, by adding `first_name` to
our query without including it in the `GROUP BY` statement, the database doesn't
know _which_ `first_name` to display in the query.

To fix this, one option is to add the column to the `GROUP BY` statement.

### Grouping by multiple columns

To fix the example above where we got an error because `first_name` is not
specified in the `GROUP BY` statement:

```sql
SELECT COUNT(*), last_name, first_name FROM Users GROUP BY last_name, first_name;

 count | last_name | first_name
-------+-----------+------------
     1 | Soap      | Jane
     1 | Doe       | Jane
     1 | Soap      | John
     1 | Doe       | John
(4 rows)
```

We now get all our rows back. This is because the combination of both
`last_name` and `first_name` will determine the groups our query will return.

In this scenario, if we had multiple users with the same `last_name` and
`first_name` we would have a row in the result that had a count greater than 1.

### Built-in aggregate functions

Most database have a number of built-in function to aggregate data.

A few commonly used ones are:

- `MIN`
- `MAX`
- `AVG`
- `SUM`
- `MEDIAN`

#### `MIN`

```sql
SELECT MIN(last_name) FROM Users;

 min
---------------
 Doe
(1 row)
```

#### `MAX`

```sql
SELECT MIN(last_name) FROM Users;
    max
------------
 2019-08-17
(1 row)
```

### Aggregate functions and multiple columns

Attempting to retrieve multiple columns with an aggregate function works in a
similar way to how the `GROUP BY` statement works; you can't retrieve a column
that's not defined in the aggregate function, because that additional column
may represent multiple values:

```sql
SELECT MAX(first_name), last_name FROM Users;

ERROR:  column "users.last_name" must appear in the GROUP BY clause or be used in an aggregate function
LINE 1: SELECT MAX(first_name), last_name FROM Users;
```

## 10. Conditionally Select out Filtered Data with SQL Where

When using the `SELECT` clause without any conditions we get all the rows in
the result.

To retrieve specific rows we use the `WHERE` clause:

```sql
SELECT * FROM Users WHERE last_name = 'Doe';

 create_date |             user_handle              | last_name | first_name
-------------+--------------------------------------+-----------+------------
 2019-08-17  | c85b174b-cc34-4948-b5c8-7acdf44ceb0f | Doe       | Jane
 2019-08-17  | cc7f43e0-0bb3-4df6-9cbc-19f99f2dd82c | Doe       | John
(2 rows)
```

One can think of the `SELECT` clause as a `for` loop, and the `WHERE` clause as
a filter.

### Conjunctions and disjunctions

We can use `AND` and `OR` to make our queries more specific:

```sql
SELECT * FROM Users WHERE last_name = 'Doe' AND 'first_name' = 'John';

 create_date |             user_handle              | last_name | first_name
-------------+--------------------------------------+-----------+------------
 2019-08-17  | cc7f43e0-0bb3-4df6-9cbc-19f99f2dd82c | Doe       | John
(1 row)

--

SELECT * FROM Users WHERE last_name = 'Doe' OR 'first_name' = 'John';

 create_date |             user_handle              | last_name | first_name
-------------+--------------------------------------+-----------+------------
 2019-08-17  | c85b174b-cc34-4948-b5c8-7acdf44ceb0f | Doe       | Jane
 2019-08-17  | 1c662040-f4dd-4188-9a93-692bee9b2888 | Soap      | John
 2019-08-17  | cc7f43e0-0bb3-4df6-9cbc-19f99f2dd82c | Doe       | John
(3 rows)
```

### Comparison operators

The following comparison operators are available in most SQL databases:

- `>` - greater than
- `>=` - greater than or equal to
- `<` - less than
- `<=` - less than or equal to
- `=` - equal
- `<>` - not equal

```sql
SELECT * FROM Users WHERE last_name = 'Doe';

 create_date |             user_handle              | last_name | first_name
-------------+--------------------------------------+-----------+------------
 2019-08-17  | 649e9396-1cb0-43dd-a907-7dc6fd23ddc7 | Soap      | Jane
 2019-08-17  | 1c662040-f4dd-4188-9a93-692bee9b2888 | Soap      | John
(2 rows)
```

**Note:** Single quotes and double quotes perform different jobs in SQL
databases. Single quotes create text strings, double quotes create identifiers,
i.e. they can be used to identify names of tables and columns while preventing
the database from using its own lowercase conversion (avoid the use of double
quotes, as per [Don’t use double quotes in PostgreSQL](https://lerner.co.il/2013/11/30/quoting-postgresql/)).

```sql
SELECT * FROM Users WHERE last_name <> 'Doe';

 create_date |             user_handle              | last_name | first_name
-------------+--------------------------------------+-----------+------------
 2019-08-17  | 649e9396-1cb0-43dd-a907-7dc6fd23ddc7 | Soap      | Jane
 2019-08-17  | 1c662040-f4dd-4188-9a93-692bee9b2888 | Soap      | John
(2 rows)
```

### Comparison predicates

SQL databases provide a number of comparison predicates to filter queries:

predicate                           | description
-----                               | ------
`expression IS NULL`                | is null
`expression IS NOT NULL`            | is not null
`expression ISNULL`                 | is null (nonstandard syntax)
`expression NOTNULL`                | is not null (nonstandard syntax)
`a BETWEEN x AND y`                 | between
`a NOT BETWEEN x AND`               | y	not between
`a BETWEEN SYMMETRIC x AND y`       | between, after sorting the comparison values
`a NOT BETWEEN SYMMETRIC x AND y`   | not between, after sorting the comparison values
`a IS DISTINCT FROM b`              | not equal, treating null like an ordinary value
`a IS NOT DISTINCT FROM b`          | equal, treating null like an ordinary value
`boolean_expression IS TRUE`        | is true
`boolean_expression IS NOT TRUE`    | is false or unknown
`boolean_expression IS FALSE`       | is false
`boolean_expression IS NOT FALSE`   | is true or unknown
`boolean_expression IS UNKNOWN`     | is unknown
`boolean_expression IS NOT UNKNOWN` | is true or false

#### `NULL` predicate

We can also select rows where specific values are NULL. Let's drop the `NOT
NULL` constraint on `last_name`, insert a record without a `last_name`, and then query
it:

```sql
-- remove NOT NULL constraint from last_name column
ALTER TABLE Users ALTER COLUMN last_name DROP NOT NULL;

-- insert user without last_name
INSERT INTO Users
  (create_date, user_handle)
  VALUES
    (NOW(), 'bb2cefa3-a509-4b1a-82c8-e6932ab4ce46', 'Killface');

-- retrieve columns where last_name is NULL
SELECT * FROM Users WHERE last_name IS NULL;

 create_date |             user_handle              | last_name | first_name
-------------+--------------------------------------+-----------+------------
 2019-08-17  | bb2cefa3-a509-4b1a-82c8-e6932ab4ce46 |           | Killface
(1 row)
```

Inversely, we can select for rows that contain values that are not NULL:

```sql
SELECT * FROM Users WHERE last_name IS NOT NULL;

 create_date |             user_handle              | last_name | first_name
-------------+--------------------------------------+-----------+------------
 2019-08-17  | 649e9396-1cb0-43dd-a907-7dc6fd23ddc7 | Soap      | Jane
 2019-08-17  | c85b174b-cc34-4948-b5c8-7acdf44ceb0f | Doe       | Jane
 2019-08-17  | 1c662040-f4dd-4188-9a93-692bee9b2888 | Soap      | John
 2019-08-17  | cc7f43e0-0bb3-4df6-9cbc-19f99f2dd82c | Doe       | John
(4 rows)
```

The `IS` opeator needs to be used when selecting for `NULL`, as using the `=`
operator will attempt to find rows that contain the value `NULL`, instead of
returning a boolean:

```sql
SELECT * FROM Users WHERE last_name = NULL;

 create_date | user_handle | last_name | first_name
-------------+-------------+-----------+------------
(0 rows)
```

#### `BETWEEN` predicate

We can select ranges of values using the `BETWEEN` comparison predicate:

```sql
SELECT * FROM Users WHERE create_date BETWEEN '2019-08-17' AND NOW();

 create_date |             user_handle              | last_name | first_name
-------------+--------------------------------------+-----------+------------
 2019-08-17  | 649e9396-1cb0-43dd-a907-7dc6fd23ddc7 | Soap      | Jane
 2019-08-17  | c85b174b-cc34-4948-b5c8-7acdf44ceb0f | Doe       | Jane
 2019-08-17  | 1c662040-f4dd-4188-9a93-692bee9b2888 | Soap      | John
 2019-08-17  | cc7f43e0-0bb3-4df6-9cbc-19f99f2dd82c | Doe       | John
 2019-08-17  | bb2cefa3-a509-4b1a-82c8-e6932ab4ce46 |           | Killface
(5 rows)
```

**Note:** If the dates were switched so that the most recent date was before the
older date, we'd get no results:

```sql
SELECT * FROM Users WHERE create_date BETWEEN NOW() AND '2019-08-17';

 create_date |             user_handle              | last_name | first_name
-------------+--------------------------------------+-----------+------------
(0 rows)
```

### Row and array comparison

SQL databases provide a number of row and array comparison clauses:

clause   | syntax
-------- | -------
`IN`     | `expression IN (value, [, ...])`
`NOT IN` | `expression NOT IN (value, [, ...])`
`ANY`    | `expression operator ANY (array expression)`
`ALL`    | `expression operator ALL (array expression)`

#### `IN` / `NOT IN`

#### `ANY` / `SOME`

#### `ALL`

