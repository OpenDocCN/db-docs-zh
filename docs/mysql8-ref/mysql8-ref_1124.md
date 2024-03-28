> 原文：[`dev.mysql.com/doc/refman/8.0/en/load-index.html`](https://dev.mysql.com/doc/refman/8.0/en/load-index.html)

#### 15.7.8.5 LOAD INDEX INTO CACHE Statement

```sql
LOAD INDEX INTO CACHE
  *tbl_index_list* [, *tbl_index_list*] ...

*tbl_index_list*:
  *tbl_name*
    [PARTITION (*partition_list*)]
    [{INDEX|KEY} (*index_name*[, *index_name*] ...)]
    [IGNORE LEAVES]

*partition_list*: {
    *partition_name*[, *partition_name*] ...
  | ALL
}
```

`LOAD INDEX INTO CACHE`语句将表索引预加载到由显式`CACHE INDEX`语句分配的关键缓存中，否则将预加载到默认关键缓存中。

`LOAD INDEX INTO CACHE`仅适用于`MyISAM`表，包括分区的`MyISAM`表。此外，可以为分区表的索引预加载一个、几个或所有分区。

`IGNORE LEAVES`修饰符仅导致预加载索引的非叶节点的块。

`IGNORE LEAVES`也支持分区`MyISAM`表。

以下语句预加载表`t1`和`t2`的索引节点（索引块）：

```sql
mysql> LOAD INDEX INTO CACHE t1, t2 IGNORE LEAVES;
+---------+--------------+----------+----------+
| Table   | Op           | Msg_type | Msg_text |
+---------+--------------+----------+----------+
| test.t1 | preload_keys | status   | OK       |
| test.t2 | preload_keys | status   | OK       |
+---------+--------------+----------+----------+
```

此语句从`t1`预加载所有索引块。它仅从`t2`预加载非叶节点的块。

`LOAD INDEX INTO CACHE`的语法允许您指定应预加载表中的哪些索引。但是，实现会将表的所有索引预加载到缓存中，因此除了表名之外，没有理由指定其他内容。

可以预加载分区`MyISAM`表的特定分区上的索引。例如，以下 2 个语句中，第一个预加载分区表`pt`的分区`p0`的索引，而第二个预加载相同表的分区`p1`和`p3`的索引：

```sql
LOAD INDEX INTO CACHE pt PARTITION (p0);
LOAD INDEX INTO CACHE pt PARTITION (p1, p3);
```

要为表`pt`中的所有分区预加载索引，您可以使用以下两个语句中的任何一个：

```sql
LOAD INDEX INTO CACHE pt PARTITION (ALL);

LOAD INDEX INTO CACHE pt;
```

刚刚显示的两个语句是等效的，发出任何一个都具有完全相同的效果。换句话说，如果您希望为分区表的所有分区预加载索引，则`PARTITION (ALL)`子句是可选的。

当为多个分区预加载索引时，分区不需要连续，并且不需要按任何特定顺序列出它们的名称。

`LOAD INDEX INTO CACHE ... IGNORE LEAVES` 除非表中所有索引具有相同的块大小，否则会失败。要确定表的索引块大小，请使用**myisamchk -dv**并检查`Blocksize`列。
