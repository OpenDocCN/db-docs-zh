# 10.3.10 索引扩展的使用

> 原文：[`dev.mysql.com/doc/refman/8.0/en/index-extensions.html`](https://dev.mysql.com/doc/refman/8.0/en/index-extensions.html)

`InnoDB`会自动通过附加主键列来扩展每个辅助索引。考虑这个表定义：

```sql
CREATE TABLE t1 (
  i1 INT NOT NULL DEFAULT 0,
  i2 INT NOT NULL DEFAULT 0,
  d DATE DEFAULT NULL,
  PRIMARY KEY (i1, i2),
  INDEX k_d (d)
) ENGINE = InnoDB;
```

这个表在列`(i1, i2)`上定义了主键。它还在列`(d)`上定义了一个辅助索引`k_d`，但在内部`InnoDB`扩展了这个索引，并将其视为列`(d, i1, i2)`。

优化器在确定如何以及是否使用该索引时，会考虑扩展辅助索引的主键列。这可以导致更高效的查询执行计划和更好的性能。

优化器可以使用扩展的辅助索引进行`ref`、`range`和`index_merge`索引访问，进行松散索引扫描访问，进行连接和排序优化，以及进行`MIN()`/`MAX()`优化。

以下示例展示了优化器是否使用扩展辅助索引会如何影响执行计划。假设`t1`被这些行填充：

```sql
INSERT INTO t1 VALUES
(1, 1, '1998-01-01'), (1, 2, '1999-01-01'),
(1, 3, '2000-01-01'), (1, 4, '2001-01-01'),
(1, 5, '2002-01-01'), (2, 1, '1998-01-01'),
(2, 2, '1999-01-01'), (2, 3, '2000-01-01'),
(2, 4, '2001-01-01'), (2, 5, '2002-01-01'),
(3, 1, '1998-01-01'), (3, 2, '1999-01-01'),
(3, 3, '2000-01-01'), (3, 4, '2001-01-01'),
(3, 5, '2002-01-01'), (4, 1, '1998-01-01'),
(4, 2, '1999-01-01'), (4, 3, '2000-01-01'),
(4, 4, '2001-01-01'), (4, 5, '2002-01-01'),
(5, 1, '1998-01-01'), (5, 2, '1999-01-01'),
(5, 3, '2000-01-01'), (5, 4, '2001-01-01'),
(5, 5, '2002-01-01');
```

现在考虑这个查询：

```sql
EXPLAIN SELECT COUNT(*) FROM t1 WHERE i1 = 3 AND d = '2000-01-01'
```

执行计划取决于是否使用了扩展索引。

当优化器不考虑索引扩展时，它将索引`k_d`视为仅`(d)`。查询的`EXPLAIN`结果如下：

```sql
mysql> EXPLAIN SELECT COUNT(*) FROM t1 WHERE i1 = 3 AND d = '2000-01-01'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
         type: ref
possible_keys: PRIMARY,k_d
          key: k_d
      key_len: 4
          ref: const
         rows: 5
        Extra: Using where; Using index
```

当优化器考虑索引扩展时，它将`k_d`视为`(d, i1, i2)`。在这种情况下，它可以使用最左边的索引前缀`(d, i1)`来生成更好的执行计划：

```sql
mysql> EXPLAIN SELECT COUNT(*) FROM t1 WHERE i1 = 3 AND d = '2000-01-01'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
         type: ref
possible_keys: PRIMARY,k_d
          key: k_d
      key_len: 8
          ref: const,const
         rows: 1
        Extra: Using index
```

在两种情况下，`key`指示优化器使用了辅助索引`k_d`，但`EXPLAIN`输出显示了使用扩展索引带来的这些改进：

+   `key_len`从 4 字节增加到 8 字节，表示键查找使用了列`d`和`i1`，而不仅仅是`d`。

+   `ref`值从`const`变为`const,const`，因为键查找使用了两个键部分，而不是一个。

+   `rows`计数从 5 减少到 1，表示`InnoDB`应该需要检查更少的行来生成结果。

+   `Extra`值从`Using where; Using index`变为`Using index`。这意味着可以仅使用索引读取行，而不需要查询数据行中的列。

使用扩展索引的优化器行为差异也可以通过`SHOW STATUS`看到：

```sql
FLUSH TABLE t1;
FLUSH STATUS;
SELECT COUNT(*) FROM t1 WHERE i1 = 3 AND d = '2000-01-01';
SHOW STATUS LIKE 'handler_read%'
```

上述语句包括`FLUSH TABLES`和`FLUSH STATUS`来刷新表缓存和清除状态计数器。

没有索引扩展，`SHOW STATUS`产生以下结果：

```sql
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Handler_read_first    | 0     |
| Handler_read_key      | 1     |
| Handler_read_last     | 0     |
| Handler_read_next     | 5     |
| Handler_read_prev     | 0     |
| Handler_read_rnd      | 0     |
| Handler_read_rnd_next | 0     |
+-----------------------+-------+
```

使用索引扩展，`SHOW STATUS` 会产生这样的结果。`Handler_read_next` 的值从 5 减少到 1，表明索引的使用更加高效：

```sql
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Handler_read_first    | 0     |
| Handler_read_key      | 1     |
| Handler_read_last     | 0     |
| Handler_read_next     | 1     |
| Handler_read_prev     | 0     |
| Handler_read_rnd      | 0     |
| Handler_read_rnd_next | 0     |
+-----------------------+-------+
```

`optimizer_switch` 系统变量的 `use_index_extensions` 标志允许控制优化器在确定如何使用 `InnoDB` 表的次要索引时是否考虑主键列。默认情况下，`use_index_extensions` 是启用的。要检查禁用索引扩展是否可以提高性能，请使用以下语句：

```sql
SET optimizer_switch = 'use_index_extensions=off';
```

优化器对索引扩展的使用受到索引中关键部分数量（16）和最大键长度（3072 字节）的通常限制。
