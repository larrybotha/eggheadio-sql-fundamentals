# SQL Fundamentals

Notes and annotations from Egghead's SQL Fundamentals course: [https://egghead.io/courses/sql-fundamentals](https://egghead.io/courses/sql-fundamentals)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [2. Create a Table with SQL Create](#2-create-a-table-with-sql-create)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 2. Create a Table with SQL Create

```bash
postgres=# CREATE TABLE Users (
  create_date date,
  user_handle uuid,
  last_name text,
  first_name text
);
```

To view tables in the database:

```bash
postgres=# \d
```

To view a specific table

```bash
postgres=# \d Users
```

Postgres has a number of different [data types](https://www.postgresql.org/docs/9.5/datatype.html)
to learn about.
