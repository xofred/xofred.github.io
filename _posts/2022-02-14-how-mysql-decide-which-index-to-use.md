---
title:  "How MySQL decide which index to use"
tags: mysql backend
toc: true

---

*No tldr this time -_-!!!*

### Concepts

- Without an index, MySQL must begin with the first row and then read through the entire table

- Most MySQL indexes (`PRIMARY KEY`, `UNIQUE`, `INDEX`, and `FULLTEXT`) are stored in [B-trees](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_b_tree)

  - Sorted

  - Fast lookup for exact matches (`=` operator) 

  - Fast lookup for ranges (for example, `>`, `<`, and `BETWEEN` operators)

  - Available for most storage engines, such as [`InnoDB`](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html) and [`MyISAM`](https://dev.mysql.com/doc/refman/8.0/en/myisam-storage-engine.html)

### Flight Manual

- Matching a `WHERE` clause

- For multiple indexes, uses the index that finds the **smallest number of rows**

- For multiple-column index, uses **leftmost prefix**. For example, if you have a three-column index on `(col1, col2, col3)`, you have indexed search capabilities on `(col1)`, `(col1, col2)`, and `(col1, col2, col3)`

- Involving comparison

  - Table `joins`, **more efficiently** if columns are the same type and size. In this context, [`VARCHAR`](https://dev.mysql.com/doc/refman/8.0/en/char.html) and [`CHAR`](https://dev.mysql.com/doc/refman/8.0/en/char.html) are considered the same
  - String columns, should use the same character set. For example, comparing a `utf8` column with a `latin1` column **precludes use of an index**
  - Dissimilar columns (comparing a string column to a temporal or numeric column, for example) **may prevent use of indexes** if values cannot be compared directly without conversion. For a given value such as `1` in the numeric column, it might compare equal to any number of values in the string column such as `'1'`, `' 1'`, `'00001'`, or `'01.e1'`. This rules out use of any indexes for the string column

- [`MIN()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_min) or [`MAX()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_max) value

- To sort or group a table if the sorting or grouping is done on a **leftmost prefix** of a usable index

- [Covering index](http://stackoverflow.com/questions/62137/ddg#62140)

  - For instance, this:

    ```
    SELECT *
    FROM tablename
    WHERE criteria
    ```

    will typically use indexes to speed up the resolution of which rows to retrieve using *criteria*, but then it will go to the full table to retrieve the rows.

    However, if the **index** contained the columns *column1, column2* and *column3*, then this sql:

    ```
    SELECT column1, column2
    FROM tablename
    WHERE criteria
    ```

    and, provided that particular **index** could be used to speed up the resolution of which rows to retrieve, the **index** already contains the values of the columns you're interested in, so it won't have to go to the table to retrieve the rows, but can produce the results directly from the **index**.

### References

[MySQL :: MySQL 8.0 Reference Manual :: 8.3.1 How MySQL Uses Indexes](https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html)

[What is a Covered Index](http://stackoverflow.com/questions/62137/ddg#62140)

[Speed up your queries using the covering index in MySQL](https://blog.toadworld.com/2017/04/06/speed-up-your-queries-using-the-covering-index-in-mysql)
