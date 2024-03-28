> 原文：[`dev.mysql.com/doc/refman/8.0/en/index-condition-pushdown-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/index-condition-pushdown-optimization.html)

#### 10.2.1.6 索引条件推送优化

索引条件推送（ICP）是一种优化，用于 MySQL 使用索引从表中检索行的情况。没有 ICP，存储引擎遍历索引以定位基表中的行，并将它们返回给 MySQL 服务器，MySQL 服务器评估行的`WHERE`条件。启用 ICP 后，如果`WHERE`条件的部分可以仅通过索引列进行评估，MySQL 服务器将此部分`WHERE`条件向下推送到存储引擎。然后存储引擎通过使用索引条目评估推送的索引条件，仅当满足条件时才从表中读取行。ICP 可以减少存储引擎必须访问基表的次数，以及 MySQL 服务器必须访问存储引擎的次数。

索引条件推送优化的适用性取决于以下条件：

+   当需要访问完整表行时，ICP 用于`range`、`ref`、`eq_ref`和`ref_or_null`访问方法。

+   ICP 可用于`InnoDB`和`MyISAM`表，包括分区的`InnoDB`和`MyISAM`表。

+   对于`InnoDB`表，ICP 仅用于辅助索引。ICP 的目标是减少完整行读取的次数，从而减少 I/O 操作。对于`InnoDB`聚簇索引，完整记录已经读入`InnoDB`缓冲区。在这种情况下使用 ICP 不会减少 I/O。

+   ICP 不支持在虚拟生成列上创建的辅助索引。`InnoDB`支持虚拟生成列上的辅助索引。

+   不能向下推送涉及子查询的条件。

+   不能向涉及存储函数的条件推送。存储引擎无法调用存储函数。

+   触发条件无法向下推送。（有关触发条件的信息，请参见第 10.2.2.3 节，“使用 EXISTS 策略优化子查询”。）

+   （*MySQL 8.0.30 及更高版本*：）条件无法向包含对系统变量引用的派生表推送。

要了解这种优化如何工作，首先考虑当未使用索引条件推送优化时索引扫描的进行方式：

1.  首先通过读取索引元组获取下一行，然后使用索引元组定位并读取完整的表行。

1.  测试适用于此表的`WHERE`条件部分。根据测试结果接受或拒绝行。

使用索引条件下推，扫描如下进行：

1.  获取下一行的索引元组（但不是完整的表行）。

1.  测试适用于此表且仅可以使用索引列进行检查的`WHERE`条件的部分。如果条件不满足，则继续到下一行的索引元组。

1.  如果条件满足，使用索引元组定位并读取完整的表行。

1.  测试适用于此表的`WHERE`条件的剩余部分。根据测试结果接受或拒绝行。

当使用索引条件下推时，`EXPLAIN`输出在`Extra`列中显示`Using index condition`。它不显示`Using index`，因为当必须读取完整的表行时，这不适用。

假设一个表包含有关人员及其地址的信息，并且该表具有定义为`INDEX (zipcode, lastname, firstname)`的索引。如果我们知道一个人的`zipcode`值，但不确定姓氏，我们可以这样搜索：

```sql
SELECT * FROM people
  WHERE zipcode='95054'
  AND lastname LIKE '%etrunia%'
  AND address LIKE '%Main Street%';
```

MySQL 可以使用索引扫描具有`zipcode='95054'`的人员。第二部分（`lastname LIKE '%etrunia%'`）无法用于限制必须扫描的行数，因此没有索引条件下推，此查询必须检索所有具有`zipcode='95054'`的人员的完整表行。

使用索引条件下推，MySQL 在读取完整的表行之前检查`lastname LIKE '%etrunia%'`部分。这避免了读取与`zipcode`条件匹配但不匹配`lastname`条件的索引元组对应的完整行。

索引条件下推默认启用。可以通过设置`optimizer_switch`系统变量来控制，设置`index_condition_pushdown`标志：

```sql
SET optimizer_switch = 'index_condition_pushdown=off';
SET optimizer_switch = 'index_condition_pushdown=on';
```

参见第 10.9.2 节，“可切换优化”。
