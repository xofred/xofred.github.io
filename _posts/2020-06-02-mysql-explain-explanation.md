---
title:  "MySQL EXPLAIN Explanation"
tags: sql mysql optimization
toc: true
toc_label: "Table of Content"
toc_icon: "cog"

---

*EXPLAIN is a useful tool for SQL query optimization*

## Here comes the problem

Let's say we have a crazy slow SQL query. We analyze it with EXPLAIN.

```sql
EXPLAIN SELECT   b.`purchase_order_id`,SUM(b.`quantity`*b.`unit_price`) posted_value FROM `rolling_price_logs` a   JOIN `purchase_order_bill_items` b    ON a.`purchase_order_bill_item_id` = b.id in ('39176','40261','40760','40977','40985','40986','41080','42355','42519','42845','42948','42979','43101','43114','43117','43118','43164','43165','43198','43203') GROUP BY b.`purchase_order_id`\G
```

We can put a `\G` in the end. It means result in a human-friendly JSON format. Instead of the default table format, which could be hard to read, unless you have a super widescreen.

```
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: b
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 250055
     filtered: 100.00
        Extra: Using temporary; Using filesort
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: a
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1690834
     filtered: 10.00
        Extra: Using where; Using join buffer (Block Nested Loop)
2 rows in set, 1 warning (0.00 sec)
```

But what does it mean? And how do we optimize our query from it?

## EXPLAIN Output Columns( from [MySQL reference](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html) )

| Column                                                       | JSON Name       | Meaning                                        |
| ------------------------------------------------------------ | --------------- | ---------------------------------------------- |
| [`id`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_id) | `select_id`     | The `SELECT` identifier                        |
| [`select_type`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_select_type) | None            | The `SELECT` type                              |
| [`table`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_table) | `table_name`    | The table for the output row                   |
| [`partitions`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_partitions) | `partitions`    | The matching partitions                        |
| [`type`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_type) | `access_type`   | The join type                                  |
| [`possible_keys`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_possible_keys) | `possible_keys` | The possible indexes to choose                 |
| [`key`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_key) | `key`           | The index actually chosen                      |
| [`key_len`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_key_len) | `key_length`    | The length of the chosen key                   |
| [`ref`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_ref) | `ref`           | The columns compared to the index              |
| [`rows`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_rows) | `rows`          | Estimate of rows to be examined                |
| [`filtered`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_filtered) | `filtered`      | Percentage of rows filtered by table condition |
| [`Extra`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_extra) | None            | Additional information                         |

### `rows` and `filtered`

Given the 1690834 `rows` and 10 percents `filtered`, we know that there are 1690834 * 0.9 = 1,521,750 rows are still need to be looked up by the database. But what's the cause?

### `possible_keys` and `key`

`NULL` means no index would be used during the query. And yes it's often the main cause for a crazy slow query, which could force the database to do a full-table scan, as the `rows` and `filtered` show.

```sql
SHOW INDEX FROM rolling_price_logs;
```

```
+--------------------+------------+----------------------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table              | Non_unique | Key_name                               | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+--------------------+------------+----------------------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| rolling_price_logs |          0 | PRIMARY                                |            1 | id          | A         |     1690834 |     NULL | NULL   |      | BTREE      |         |               |
| rolling_price_logs |          1 | index_rolling_price_logs_on_product_id |            1 | product_id  | A         |       30823 |     NULL | NULL   | YES  | BTREE      |         |               |
+--------------------+------------+----------------------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.00 sec)
```

As we check the indices of table a, the join column `purchase_order_bill_item_id` which happens to be the foreign key of table b, has no index on it. We should add a index on it.

```sql
ALTER TABLE rolling_price_logs ADD INDEX index_rolling_price_logs_on_purchase_order_bill_item_id (purchase_order_bill_item_id);
```

```
Query OK, 0 rows affected (2.28 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

After index added, we run the `EXPLAIN` again.

```
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: b
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 250055
     filtered: 100.00
        Extra: Using temporary; Using filesort
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: a
   partitions: NULL
         type: ref
possible_keys: purchase_order_bill_item_id
          key: purchase_order_bill_item_id
      key_len: 5
          ref: func
         rows: 644
     filtered: 100.00
        Extra: Using where; Using index
2 rows in set, 1 warning (0.01 sec)
```

The `rows` and `filtered` are much better. And if we execute the actual SQL query now, the database will have result instantly.

```
+-------------------+--------------+
| purchase_order_id | posted_value |
+-------------------+--------------+
|             13845 |         7.20 |
|             13897 |       242.00 |
|             13911 |       617.96 |
|             13915 |        19.60 |
|             14019 |        91.92 |
|             14094 |       402.24 |
|             14150 |        71.36 |
|             14164 |       125.92 |
+-------------------+--------------+
8 rows in set (0.43 sec)
```

## Reference

[MySQL EXPLAIN Output Format](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)
