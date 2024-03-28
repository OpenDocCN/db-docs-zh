> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-open-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/show-open-tables.html)

#### 15.7.7.24 SHOW OPEN TABLES 语句

```sql
SHOW OPEN TABLES
    [{FROM | IN} *db_name*]
    [LIKE '*pattern*' | WHERE *expr*]
```

`SHOW OPEN TABLES`列出了当前在表缓存中打开的非`TEMPORARY`表。请参阅 Section 10.4.3.1, “MySQL 如何打开和关闭表”。`FROM`子句（如果存在）限制显示的表为*`db_name`*数据库中存在的表。`LIKE`子句（如果存在）指示要匹配的表名。`WHERE`子句可以用于使用更一般的条件选择行，如 Section 28.8, “SHOW 语句的扩展”中所讨论的。

`SHOW OPEN TABLES`输出包含这些列：

+   `Database`

    包含表的数据库。

+   `Table`

    表名。

+   `In_use`

    表中的表锁数或锁请求数。例如，如果一个客户端使用`LOCK TABLE t1 WRITE`为表获取锁，`In_use`为 1。如果另一个客户端在表仍被锁定时发出`LOCK TABLE t1 WRITE`，该客户端会被阻塞，等待锁，但锁请求会导致`In_use`为 2。如果计数为零，则表是打开的但当前未被使用。`In_use`也会被`HANDLER ... OPEN`语句增加，并被`HANDLER ... CLOSE`语句减少。

+   `Name_locked`

    表名是否被锁定。名称锁定用于诸如删除或重命名表等操作。

如果您对表没有权限，它在[`SHOW OPEN TABLES`](https://dev.mysql.com/doc/refman/8.0/en/show-open-tables.html)的输出中不会显示。
