> 原文：[`dev.mysql.com/doc/refman/8.0/en/nested-loop-joins.html`](https://dev.mysql.com/doc/refman/8.0/en/nested-loop-joins.html)

#### 10.2.1.7 嵌套循环连接算法

MySQL 使用嵌套循环算法或其变体来执行表之间的连接。

+   嵌套循环连接算法

+   块嵌套循环连接算法

##### 嵌套循环连接算法

简单的嵌套循环连接（NLJ）算法从第一个表中逐行读取行，将每行传递给处理连接中下一个表的嵌套循环。这个过程重复进行，直到所有要连接的表都处理完为止。

假设要使用以下连接类型执行三个表`t1`、`t2`和`t3`之间的连接：

```sql
Table   Join Type
t1      range
t2      ref
t3      ALL
```

如果使用简单的 NLJ 算法，则连接的处理方式如下：

```sql
for each row in t1 matching range {
  for each row in t2 matching reference key {
    for each row in t3 {
      if row satisfies join conditions, send to client
    }
  }
}
```

因为 NLJ 算法将行逐个从外部循环传递到内部循环，所以通常会多次读取在内部循环中处理的表。

##### 块嵌套循环连接算法

块嵌套循环（BNL）连接算法使用缓冲区来减少内部循环中读取表的次数。例如，如果将 10 行读入缓冲区并将缓冲区传递给下一个内部循环，那么内部循环中读取的每一行都可以与缓冲区中的所有 10 行进行比较。这将使内部表的读取次数减少一个数量级。

在 MySQL 8.0.18 之前，当无法使用索引时，此算法用于等值连接；在 MySQL 8.0.18 及更高版本中，在这种情况下使用哈希连接优化。从 MySQL 8.0.20 开始，MySQL 不再使用块嵌套循环，并且在以前使用块嵌套循环的所有情况下都使用哈希连接。参见第 10.2.1.4 节“哈希连接优化”。

MySQL 连接缓冲具有以下特点：

+   当连接的类型为`ALL`或`index`（换句话说，无法使用任何可能的键，并且需要进行完全扫描，无论是数据行还是索引行），或者`range`时，可以使用连接缓冲。连接缓冲也适用于外连接，如第 10.2.1.12 节“块嵌套循环和批量键访问连接”中所述。

+   为第一个非常量表不分配连接缓冲区，即使它的类型为`ALL`或`index`。

+   仅将连接中感兴趣的列存储在其连接缓冲区中，而不是整行。

+   `join_buffer_size` 系统变量确定用于处理查询的每个连接缓冲区的大小。

+   为每个可以缓冲的连接分配一个缓冲区，因此一个给定的查询可能会使用多个连接缓冲区。

+   在执行连接之前分配连接缓冲区，并在查询完成后释放。

对于之前描述的 NLJ 算法的示例连接（不使用缓冲），使用连接缓冲区进行连接如下：

```sql
for each row in t1 matching range {
  for each row in t2 matching reference key {
    store used columns from t1, t2 in join buffer
    if buffer is full {
      for each row in t3 {
        for each t1, t2 combination in join buffer {
          if row satisfies join conditions, send to client
        }
      }
      empty join buffer
    }
  }
}

if buffer is not empty {
  for each row in t3 {
    for each t1, t2 combination in join buffer {
      if row satisfies join conditions, send to client
    }
  }
}
```

如果 *`S`* 是连接缓冲区中每个存储的 `t1`, `t2` 组合的大小，*`C`* 是缓冲区中组合的数量，则表 `t3` 被扫描的次数为：

```sql
(*S* * *C*)/join_buffer_size + 1
```

当 `join_buffer_size` 的值增加时，`t3` 扫描的次数会减少，直到 `join_buffer_size` 足够大以容纳所有先前的行组合为止。在那一点上，通过增大它不会获得速度上的提升。
