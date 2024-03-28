> 原文：[`dev.mysql.com/doc/refman/8.0/en/row-constructor-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/row-constructor-optimization.html)

#### 10.2.1.22 行构造函数表达式优化

行构造函数允许同时比较多个值。例如，以下两个语句在语义上是等价的：

```sql
SELECT * FROM t1 WHERE (column1,column2) = (1,1);
SELECT * FROM t1 WHERE column1 = 1 AND column2 = 1;
```

此外，优化器以相同方式处理这两个表达式。

如果行构造函数的列不覆盖索引的前缀，优化器就不太可能使用可用的索引。考虑以下表，其主键为`(c1, c2, c3)`：

```sql
CREATE TABLE t1 (
  c1 INT, c2 INT, c3 INT, c4 CHAR(100),
  PRIMARY KEY(c1,c2,c3)
);
```

在此查询中，`WHERE`子句使用索引中的所有列。然而，行构造函数本身不覆盖索引前缀，导致优化器仅使用`c1`（`key_len=4`，即`c1`的大小）：

```sql
mysql> EXPLAIN SELECT * FROM t1
       WHERE c1=1 AND (c2,c3) > (1,1)\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 3
     filtered: 100.00
        Extra: Using where
```

在这种情况下，将行构造函数表达式重写为等效的非构造函数表达式可能会导致更完整的索引使用。对于给定的查询，行构造函数和等效的非构造函数表达式分别为：

```sql
(c2,c3) > (1,1)
c2 > 1 OR ((c2 = 1) AND (c3 > 1))
```

将查询重写为使用非构造函数表达式会导致优化器在索引中使用所有三列（`key_len=12`）：

```sql
mysql> EXPLAIN SELECT * FROM t1
       WHERE c1 = 1 AND (c2 > 1 OR ((c2 = 1) AND (c3 > 1)))\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: range
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 12
          ref: NULL
         rows: 3
     filtered: 100.00
        Extra: Using where
```

因此，为了获得更好的结果，避免将行构造函数与`AND`/`OR`表达式混合使用。选择其中一个使用。

在某些条件下，优化器可以对具有行构造函数参数的`IN()`表达式应用范围访问方法。参见行构造函数表达式的范围优化。
