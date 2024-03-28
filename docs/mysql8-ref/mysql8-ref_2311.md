> 原文：[`dev.mysql.com/doc/refman/8.0/en/cannot-find-table.html`](https://dev.mysql.com/doc/refman/8.0/en/cannot-find-table.html)

#### B.3.2.14 表 'tbl_name' 不存在

如果出现以下任一错误，则通常意味着默认数据库中不存在具有给定名称的表：

```sql
Table '*tbl_name*' doesn't exist
Can't find file: '*tbl_name*' (errno: 2)
```

在某些情况下，可能是表确实存在，但您引用它的方式不正确：

+   因为 MySQL 使用目录和文件来存储数据库和表，如果它们位于区分大小写的文件名的文件系统上，则数据库和表名称区分大小写。

+   即使对于不区分大小写的文件系统，例如 Windows，查询中对给定表的所有引用也必须使用相同的大小写形式。

您可以使用 `SHOW TABLES` 命令查看默认数据库中有哪些表。参见 Section 15.7.7, “SHOW Statements”。
