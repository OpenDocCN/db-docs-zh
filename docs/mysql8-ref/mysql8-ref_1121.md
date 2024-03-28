> 原文：[`dev.mysql.com/doc/refman/8.0/en/cache-index.html`](https://dev.mysql.com/doc/refman/8.0/en/cache-index.html)

#### 15.7.8.2 缓存索引语句

```sql
CACHE INDEX {
      *tbl_index_list* [, *tbl_index_list*] ...
    | *tbl_name* PARTITION (*partition_list*)
  }
  IN *key_cache_name*

*tbl_index_list*:
  *tbl_name* [{INDEX|KEY} (*index_name*[, *index_name*] ...)]

*partition_list*: {
    *partition_name*[, *partition_name*] ...
  | ALL
}
```

[`缓存索引`](https://dev.mysql.com/doc/refman/8.0/en/cache-index.html)语句将表索引分配给特定的键缓存。它仅适用于`MyISAM`表，包括分区的`MyISAM`表。在索引被分配后，如果需要，它们可以通过`加载索引到缓存`进行预加载。

以下语句将表`t1`、`t2`和`t3`的索引分配给名为`hot_cache`的键缓存：

```sql
mysql> CACHE INDEX t1, t2, t3 IN hot_cache;
+---------+--------------------+----------+----------+
| Table   | Op                 | Msg_type | Msg_text |
+---------+--------------------+----------+----------+
| test.t1 | assign_to_keycache | status   | OK       |
| test.t2 | assign_to_keycache | status   | OK       |
| test.t3 | assign_to_keycache | status   | OK       |
+---------+--------------------+----------+----------+
```

[`缓存索引`](https://dev.mysql.com/doc/refman/8.0/en/cache-index.html)的语法允许您指定只有特定索引应该分配给缓存。然而，实现会将表的所有索引分配给缓存，因此除了表名外，没有理由指定其他内容。

在[`缓存索引`](https://dev.mysql.com/doc/refman/8.0/en/cache-index.html)语句中引用的键缓存可以通过参数设置语句或服务器参数设置来创建其大小。例如：

```sql
SET GLOBAL keycache1.key_buffer_size=128*1024;
```

键缓存参数作为结构化系统变量的成员访问。参见第 7.1.9.5 节，“结构化系统变量”。

在分配索引之前，必须存在一个键缓存，否则会出错：

```sql
mysql> CACHE INDEX t1 IN non_existent_cache;
ERROR 1284 (HY000): Unknown key cache 'non_existent_cache'
```

默认情况下，表索引分配给在服务器启动时创建的主（默认）键缓存。当键缓存被销毁时，分配给它的所有索引将重新分配给默认键缓存。

索引分配影响全局服务器：如果一个客户端将索引分配给给定的缓存，那么无论哪个客户端发出查询，该缓存都用于涉及该索引的所有查询。

[`缓存索引`](https://dev.mysql.com/doc/refman/8.0/en/cache-index.html)支持分区的`MyISAM`表。您可以将一个或多个索引分配给一个给定的键缓存的一个、几个或所有分区。例如，您可以执行以下操作：

```sql
CREATE TABLE pt (c1 INT, c2 VARCHAR(50), INDEX i(c1))
    ENGINE=MyISAM
    PARTITION BY HASH(c1)
    PARTITIONS 4;

SET GLOBAL kc_fast.key_buffer_size = 128 * 1024;
SET GLOBAL kc_slow.key_buffer_size = 128 * 1024;

CACHE INDEX pt PARTITION (p0) IN kc_fast;
CACHE INDEX pt PARTITION (p1, p3) IN kc_slow;
```

前一组语句执行以下操作：

+   创建一个具有 4 个分区的分区表；这些分区自动命名为`p0`，...，`p3`；此表在列`c1`上有一个名为`i`的索引。

+   创建名为`kc_fast`和`kc_slow`的 2 个键缓存

+   将分区`p0`的索引分配给`kc_fast`键缓存，将分区`p1`和`p3`的索引分配给`kc_slow`键缓存；剩余分区（`p2`）的索引使用服务器的默认键缓存。

如果您希望将表`pt`中所有分区的索引分配给名为`kc_all`的单个键缓存，可以使用以下两个语句之一：

```sql
CACHE INDEX pt PARTITION (ALL) IN kc_all;

CACHE INDEX pt IN kc_all;
```

刚刚展示的两个语句是等效的，发出任何一个都会产生完全相同的效果。换句话说，如果您希望为分区表的所有分区分配索引到同一个键缓存中，`PARTITION (ALL)` 子句是可选的。

当为多个分区分配索引到一个键缓存时，这些分区不需要是连续的，也不需要按任何特定顺序列出它们的名称。未明确分配到键缓存的任何分区的索引将自动使用服务器默认的键缓存。

对于分区的`MyISAM`表也支持索引预加载。有关更多信息，请参见第 15.7.8.5 节，“LOAD INDEX INTO CACHE Statement”。
