# 15.1.27 DROP INDEX 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-index.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-index.html)

```sql
DROP INDEX *index_name* ON *tbl_name*
    [*algorithm_option* | *lock_option*] ...

*algorithm_option*:
    ALGORITHM [=] {DEFAULT | INPLACE | COPY}

*lock_option*:
    LOCK [=] {DEFAULT | NONE | SHARED | EXCLUSIVE}
```

`DROP INDEX` 从表 *`tbl_name`* 中删除名为 *`index_name`* 的索引。此语句被映射为一个 `ALTER TABLE` 语句来删除索引。请参见 Section 15.1.9, “ALTER TABLE Statement”。

要删除主键，索引名称始终为 `PRIMARY`，必须将其指定为带引号的标识符，因为 `PRIMARY` 是一个保留字：

```sql
DROP INDEX `PRIMARY` ON t;
```

对于`NDB`表中的可变宽度列的索引是在线删除的；也就是说，不需要进行任何表复制。尽管表在操作期间针对相同的 API 节点被锁定，但不会阻止其他 NDB Cluster API 节点访问该表。服务器会在确定可能进行此操作时自动执行；您不需要使用任何特殊的 SQL 语法或服务器选项来触发此操作。

`ALGORITHM` 和 `LOCK` 子句可以用来影响表复制方法和读写表时的并发级别，当其索引正在被修改时。它们与`ALTER TABLE`语句具有相同的含义。更多信息，请参见 Section 15.1.9, “ALTER TABLE Statement”

MySQL NDB Cluster 支持使用标准 MySQL Server 中支持的相同 `ALGORITHM=INPLACE` 语法进行在线操作。更多信息，请参见 Section 25.6.12, “Online Operations with ALTER TABLE in NDB Cluster”。
