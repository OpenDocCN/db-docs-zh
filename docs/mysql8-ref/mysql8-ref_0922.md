# 15.1.24 DROP DATABASE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-database.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-database.html)

```sql
DROP {DATABASE | SCHEMA} [IF EXISTS] *db_name*
```

`DROP DATABASE`会删除数据库中的所有表并删除数据库。对于这个语句要*非常*小心！要使用`DROP DATABASE`，您需要在数据库上拥有`DROP`权限。`DROP SCHEMA`是`DROP DATABASE`的同义词。

重要提示

当一个数据库被删除时，为该数据库专门授予的权限*不会*自动删除。必须手动删除它们。参见 Section 15.7.1.6, “GRANT Statement”。

`IF EXISTS`用于防止数据库不存在��发生错误。

如果默认数据库被删除，那么默认数据库将被取消设置（`DATABASE()`函数返回`NULL`）。

如果你在一个符号链接的数据库上使用`DROP DATABASE`，那么链接和原始数据库都会被删除。

`DROP DATABASE`会返回被删除的表的数量。

`DROP DATABASE`语句会从给定的数据库目录中删除 MySQL 在正常操作期间可能创建的文件和目录。这包括以下列表中显示的所有带有扩展名的文件：

+   `.BAK`

+   `.DAT`

+   `.HSH`

+   `.MRG`

+   `.MYD`

+   `.MYI`

+   `.cfg`

+   `.db`

+   `.ibd`

+   `.ndb`

如果在 MySQL 删除了刚列出的文件后，数据库目录中仍然存在其他文件或目录，则无法删除数据库目录。在这种情况下，您必须手动删除任何剩余的文件或目录，并再次发出`DROP DATABASE`语句。

删除数据库不会移除在该数据库中创建的任何`TEMPORARY`表。`TEMPORARY`表在创建它们的会话结束时会自动删除。参见 Section 15.1.20.2, “CREATE TEMPORARY TABLE Statement”。

你也可以使用**mysqladmin**来删除数据库。参见 Section 6.5.2, “mysqladmin — A MySQL Server Administration Program”。
