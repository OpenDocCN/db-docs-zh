# 10.3.9 B-树和哈希索引的比较

> 原文：[`dev.mysql.com/doc/refman/8.0/en/index-btree-hash.html`](https://dev.mysql.com/doc/refman/8.0/en/index-btree-hash.html)

理解 B-树和哈希数据结构可以帮助预测不同查询在使用这些数据结构的不同存储引擎上的性能表现，特别是对于允许选择 B-树或哈希索引的 `MEMORY` 存储引擎。

+   B-树索引特性

+   哈希索引特性

#### B-树索引特性

B-树索引可用于使用 `=`, `>`, `>=`, `<`, `<=`, 或 `BETWEEN` 操作符的表达式中的列比较。如果 `LIKE` 的参数是不以通配符字符开头的常量字符串，则也可用于 `LIKE` 比较。例如，以下 `SELECT` 语句使用索引：

```sql
SELECT * FROM *tbl_name* WHERE *key_col* LIKE 'Patrick%';
SELECT * FROM *tbl_name* WHERE *key_col* LIKE 'Pat%_ck%';
```

在第一条语句中，只考虑 `'Patrick' <= *`key_col`* < 'Patricl'` 的行。在第二条语句中，只考虑 `'Pat' <= *`key_col`* < 'Pau'` 的行。

以下 `SELECT` 语句不使用索引：

```sql
SELECT * FROM *tbl_name* WHERE *key_col* LIKE '%Patrick%';
SELECT * FROM *tbl_name* WHERE *key_col* LIKE *other_col*;
```

在第一条语句中，`LIKE` 值以通配符字符开头。在第二条语句中，`LIKE` 值不是一个常量。

如果使用 `... LIKE '%*`string`*%'`，且 *`string`* 长度超过三个字符，MySQL 将使用 Turbo Boyer-Moore 算法初始化字符串的模式，然后使用该模式更快地执行搜索。

使用 `*`col_name`* IS NULL` 进行搜索时，如果 *`col_name`* 被索引，则会使用索引。

任何不跨越 `WHERE` 子句中的所有 `AND` 级别的索引都不会被用来优化查询。换句话说，为了能够使用索引，索引的前缀必须在每个 `AND` 组中被使用。

以下 `WHERE` 子句使用索引：

```sql
... WHERE *index_part1*=1 AND *index_part2*=2 AND *other_column*=3

    /* *index* = 1 OR *index* = 2 */
... WHERE *index*=1 OR A=10 AND *index*=2

    /* optimized like "*index_part1*='hello'" */
... WHERE *index_part1*='hello' AND *index_part3*=5

    /* Can use index on *index1* but not on *index2* or *index3* */
... WHERE *index1*=1 AND *index2*=2 OR *index1*=3 AND *index3*=3;
```

这些 `WHERE` 子句 *不* 使用索引：

```sql
 /* *index_part1* is not used */
... WHERE *index_part2*=1 AND *index_part3*=2

    /*  Index is not used in both parts of the WHERE clause  */
... WHERE *index*=1 OR A=10

    /* No index spans all rows  */
... WHERE *index_part1*=1 OR *index_part2*=10
```

有时候 MySQL 即使有索引也不会使用。发生这种情况的一个情况是，优化器估计使用索引需要 MySQL 访问表中非常大比例的行。在这种情况下，表扫描可能会快得多，因为它需要更少的查找。然而，如果这样的查询使用`LIMIT`只检索一些行，MySQL 仍然会使用索引，因为它可以更快地找到要返回的少数行。

#### 哈希索引特性

哈希索引具有与刚才讨论的索引略有不同的特性：

+   它们仅用于使用`=`或`<=>`运算符的相等比较（但非常快）。它们不用于查找一系列值的比较运算符，如`<`。依赖这种单值查找的系统被称为“键值存储”；为了在这类应用中使用 MySQL，请尽可能使用哈希索引。

+   优化器无法使用哈希索引加速`ORDER BY`操作。（这种类型的索引不能用于按顺序搜索下一个条目。）

+   MySQL 无法准确确定两个值之间有多少行（这是范围优化器用来决定使用哪个索引的）。如果你将一个`MyISAM`或`InnoDB`表更改为哈希索引的`MEMORY`表，这可能会影响一些查询。

+   只有整个键才能用于搜索行。（使用 B 树索引，任何键的最左前缀都可以用于查找行。）
