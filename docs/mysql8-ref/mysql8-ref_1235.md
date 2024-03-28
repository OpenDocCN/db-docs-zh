# 17.11.4 表碎片整理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-file-defragmenting.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-file-defragmenting.html)

随机插入或从二级索引中删除可能导致索引碎片化。碎片化意味着磁盘上索引页面的物理排序与页面上记录的索引排序不接近，或者在为索引分配的 64 页块中有许多未使用的页面。

碎片化的一个症状是表占用的空间比“应该”占用的空间多。确切的数量很难确定。所有 `InnoDB` 数据和索引都存储在 B 树中，它们的填充因子可能从 50% 变化到 100%。碎片化的另一个症状是像这样的表扫描所需的时间比“应该”花费的时间更长：

```sql
SELECT COUNT(*) FROM t WHERE *non_indexed_column* <> 12345;
```

前面的查询需要 MySQL 执行完整表扫描，这是针对大表最慢的查询类型。

为加快索引扫描速度，您可以定期执行“null”`ALTER TABLE`操作，这会导致 MySQL 重建表：

```sql
ALTER TABLE *tbl_name* ENGINE=INNODB
```

您还可以使用`ALTER TABLE *`tbl_name`* FORCE`执行“null” alter 操作，重建表。

`ALTER TABLE *`tbl_name`* ENGINE=INNODB` 和 `ALTER TABLE *`tbl_name`* FORCE` 都使用在线 DDL。有关更多信息，请参见 Section 17.12, “InnoDB and Online DDL”。

另一种执行碎片整理操作的方法是使用**mysqldump**将表转储到文本文件中，删除表，并从转储文件重新加载。

如果对索引的插入始终是升序的，并且仅从末尾删除记录，则 `InnoDB` 文件空间管理算法保证索引不会发生碎片化。
