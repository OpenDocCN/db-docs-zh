> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-temporary-table.html`](https://dev.mysql.com/doc/refman/8.0/en/create-temporary-table.html)

#### 15.1.20.2 CREATE TEMPORARY TABLE Statement

在创建表时可以使用 `TEMPORARY` 关键字。`TEMPORARY` 表仅在当前会话中可见，并在会话关闭时自动删除。这意味着两个不同的会话可以使用相同的临时表名称而不会与彼此或同名的现有非 `TEMPORARY` 表发生冲突。（现有表在临时表被删除之前是隐藏的。）

`InnoDB` 不支持压缩临时表。当启用 `innodb_strict_mode`（默认情况下）时，如果指定了 `ROW_FORMAT=COMPRESSED` 或 `KEY_BLOCK_SIZE`，`CREATE TEMPORARY TABLE` 将返回错误。如果禁用了 `innodb_strict_mode`，则会发出警告并使用非压缩行格式创建临时表。`innodb_file_per-table` 选项不影响创建 `InnoDB` 临时表。

`CREATE TABLE` 会导致隐式提交，除非与 `TEMPORARY` 关键字一起使用。参见 Section 15.3.3, “Statements That Cause an Implicit Commit”。

`TEMPORARY` 表与数据库（模式）之间的关系非常松散。删除数据库不会自动删除在该数据库中创建的任何 `TEMPORARY` 表。

要创建临时表，您必须具有 `CREATE TEMPORARY TABLES` 权限。会话创建临时表后，服务器不会对表执行进一步的权限检查。创建会话可以对表执行任何操作，如 `DROP TABLE`、`INSERT`、`UPDATE` 或 `SELECT`。

这种行为的一个含义是，一个会话可以操作其临时表，即使当前用户没有创建它们的特权。假设当前用户没有`CREATE TEMPORARY TABLES`特权，但能够执行一个以定义者上下文执行且具有`CREATE TEMPORARY TABLES`特权的用户权限的存储过程，该存储过程创建了一个临时表。在存储过程执行时，会话使用定义用户的权限。存储过程返回后，有效权限会恢复到当前用户的权限，当前用户仍然可以看到临时表并对其执行任何操作。

你不能使用`CREATE TEMPORARY TABLE ... LIKE`来基于`mysql`表空间、`InnoDB`系统表空间(`innodb_system`)或通用表空间中的表的定义创建一个空表。这种表的表空间定义包括一个`TABLESPACE`属性，定义了表所在的表空间，而上述表空间不支持临时表。要基于这种表的定义创建一个临时表，使用以下语法：

```sql
CREATE TEMPORARY TABLE *new_tbl* SELECT * FROM *orig_tbl* LIMIT 0;
```

注意

自 MySQL 8.0.13 起，使用`CREATE TEMPORARY TABLE`中的`TABLESPACE = innodb_file_per_table`和`TABLESPACE = innodb_temporary`子句已被弃用；预计在未来的 MySQL 版本中将被移除。
