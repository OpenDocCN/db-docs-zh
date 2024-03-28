# 15.1.32 DROP TABLE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-table.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-table.html)

```sql
DROP [TEMPORARY] TABLE [IF EXISTS]
    *tbl_name* [, *tbl_name*] ...
    [RESTRICT | CASCADE]
```

`DROP TABLE` 可以移除一个或多个表。对于每个表，您必须拥有 `DROP` 权限。

*请谨慎* 使用这个语句！对于每个表格，它会删除表格定义和所有表格数据。如果表格被分区，该语句会删除表格定义、所有分区、存储在这些分区中的所有数据以及与被删除表格相关的所有分区定义。

删除表格也会删除表格的任何触发器。

`DROP TABLE` 会导致隐式提交，除非与 `TEMPORARY` 关键字一起使用。请参阅 Section 15.3.3, “Statements That Cause an Implicit Commit”。

重要

当删除表格时，专门为表格授予的权限 *不会* 自动删除。必须手动删除它们。请参阅 Section 15.7.1.6, “GRANT Statement”。

如果参数列表中命名的任何表格不存在，则 `DROP TABLE` 的行为取决于是否给出 `IF EXISTS` 子句：

+   没有 `IF EXISTS`，该语句将因无法删除不存在的表格而失败，并且不会进行任何更改。

+   使用 `IF EXISTS`，对于不存在的表格不会发生错误。该语句会删除所有存在的命名表格，并为每个不存在的表格生成一个 `NOTE` 诊断信息。这些注释可以通过 `SHOW WARNINGS` 显示。请参阅 Section 15.7.7.42, “SHOW WARNINGS Statement”。

在一些异常情况下，`IF EXISTS` 对于删除表格也是有用的，即在数据字典中存在条目但存储引擎中没有管理的表格的情况下。（例如，如果在从存储引擎中删除表格后但在删除数据字典条目之前发生异常服务器退出的情况。）

`TEMPORARY` 关键字具有以下效果：

+   该语句仅删除 `TEMPORARY` 表。

+   该语句不会导致隐式提交。

+   不会检查访问权限。`TEMPORARY` 表只能在创建它的会话中可见，因此不需要检查。

包含 `TEMPORARY` 关键字是防止意外删除非`TEMPORARY`表的好方法。

`RESTRICT` 和 `CASCADE` 关键字无效。它们被允许是为了更容易从其他数据库系统进行移植。

`DROP TABLE` 不支持所有 `innodb_force_recovery` 设置。请参阅 Section 17.21.3, “Forcing InnoDB Recovery”。
