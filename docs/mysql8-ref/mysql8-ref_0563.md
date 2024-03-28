# 10.3.12 不可见索引

> 原文：[`dev.mysql.com/doc/refman/8.0/en/invisible-indexes.html`](https://dev.mysql.com/doc/refman/8.0/en/invisible-indexes.html)

MySQL 支持不可见索引；也就是说，优化器不使用的索引。该功能适用于主键之外的索引（显式或隐式）。

索引默认是可见的。要明确控制新索引的可见性，可以在`CREATE TABLE`、`CREATE INDEX`或`ALTER TABLE`的索引定义中使用`VISIBLE`或`INVISIBLE`关键字：

```sql
CREATE TABLE t1 (
  i INT,
  j INT,
  k INT,
  INDEX i_idx (i) INVISIBLE
) ENGINE = InnoDB;
CREATE INDEX j_idx ON t1 (j) INVISIBLE;
ALTER TABLE t1 ADD INDEX k_idx (k) INVISIBLE;
```

要更改现有索引的可见性，请在`ALTER TABLE ... ALTER INDEX`操作中使用`VISIBLE`或`INVISIBLE`关键字：

```sql
ALTER TABLE t1 ALTER INDEX i_idx INVISIBLE;
ALTER TABLE t1 ALTER INDEX i_idx VISIBLE;
```

关于索引是可见还是不可见的信息可以从信息模式`STATISTICS`表或`SHOW INDEX`输出中获取。例如：

```sql
mysql> SELECT INDEX_NAME, IS_VISIBLE
       FROM INFORMATION_SCHEMA.STATISTICS
       WHERE TABLE_SCHEMA = 'db1' AND TABLE_NAME = 't1';
+------------+------------+
| INDEX_NAME | IS_VISIBLE |
+------------+------------+
| i_idx      | YES        |
| j_idx      | NO         |
| k_idx      | NO         |
+------------+------------+
```

不可见索引使得可以测试删除索引对查询性能的影响，而无需进行破坏性更改，如果索引被证明是必需的，则必须撤消更改。对于大表来说，删除和重新添加索引可能很昂贵，而使其不可见和可见是快速的、就地操作。

如果优化器实际上需要或使用了一个不可见的索引，有几种方法可以注意到其在表的查询中的缺失的影响：

+   包含引用不可见索引的索引提示的查询出现错误。

+   性能模式数据显示受影响查询的工作负载增加。

+   查询具有不同的`EXPLAIN`执行计划。

+   以前未出现在慢查询日志中的查询现在出现了。

`optimizer_switch`系统变量的`use_invisible_indexes`标志控制优化器是否使用不可见索引进行查询执行计划构建。如果标志是`off`（默认值），优化器会忽略不可见索引（与引入此标志之前的行为相同）。如果标志是`on`，不可见索引仍然保持不可见，但优化器会考虑它们用于执行计划构建。

使用`SET_VAR`优化提示临时更新`optimizer_switch`的值，您可以仅在单个查询的持续时间内启用不可见索引，就像这样：

```sql
mysql> EXPLAIN SELECT /*+ SET_VAR(optimizer_switch = 'use_invisible_indexes=on') */
     >     i, j FROM t1 WHERE j >= 50\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: range
possible_keys: j_idx
          key: j_idx
      key_len: 5
          ref: NULL
         rows: 2
     filtered: 100.00
        Extra: Using index condition 
mysql> EXPLAIN SELECT i, j FROM t1 WHERE j >= 50\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 5
     filtered: 33.33
        Extra: Using where
```

索引的可见性不影响索引维护。例如，索引仍会根据表行的更改而更新，唯一索引阻止重复插入到列中，无论索引是可见还是不可见。

如果表中没有明确的主键，但在`NOT NULL`列上有任何`UNIQUE`索引，则仍可能具有有效的隐式主键。在这种情况下，第一个这样的索引对表行施加与明确主键相同的约束，该索引无法隐藏。考虑以下表定义：

```sql
CREATE TABLE t2 (
  i INT NOT NULL,
  j INT NOT NULL,
  UNIQUE j_idx (j)
) ENGINE = InnoDB;
```

定义中没有明确的主键，但对`NOT NULL`列`j`上的索引会对行施加与主键相同的约束，且无法隐藏：

```sql
mysql> ALTER TABLE t2 ALTER INDEX j_idx INVISIBLE;
ERROR 3522 (HY000): A primary key index cannot be invisible.
```

现在假设在表中添加了一个明确的主键：

```sql
ALTER TABLE t2 ADD PRIMARY KEY (i);
```

明确的主键无法隐藏。此外，对`j`的唯一索引不再充当隐式主键，因此可以隐藏：

```sql
mysql> ALTER TABLE t2 ALTER INDEX j_idx INVISIBLE;
Query OK, 0 rows affected (0.03 sec)
```
