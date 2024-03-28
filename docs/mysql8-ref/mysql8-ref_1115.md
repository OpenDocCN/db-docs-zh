> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/show-tables.html)

#### 15.7.7.39 SHOW TABLES Statement

```sql
SHOW [EXTENDED] [FULL] TABLES
    [{FROM | IN} *db_name*]
    [LIKE '*pattern*' | WHERE *expr*]
```

`SHOW TABLES`列出给定数据库中的非`TEMPORARY`表。您也可以使用**mysqlshow *`db_name`***命令获取此列表。如果存在`LIKE`子句，则表示要匹配的表名。`WHERE`子句可以用于使用更一般的条件选择行，如第 28.8 节，“SHOW 语句的扩展”中所讨论的。

`LIKE`子句执行的匹配取决于`lower_case_table_names`系统变量的设置。

可选的`EXTENDED`修饰符会导致`SHOW TABLES`列出由失败的`ALTER TABLE`语句创建的隐藏表。这些临时表的名称以`#sql`开头，可以使用`DROP TABLE`进行删除。

这个语句还列出了数据库中的任何视图。可选的`FULL`修饰符会导致`SHOW TABLES`显示第二个输出列，其中表的值为`BASE TABLE`，视图的值为`VIEW`，`INFORMATION_SCHEMA`表的值为`SYSTEM VIEW`。

如果您对基表或视图没有权限，则它不会出现在`SHOW TABLES`或**mysqlshow db_name**的输出中。

表信息也可以从`INFORMATION_SCHEMA`的`TABLES`表中获取。请参阅第 28.3.38 节，“INFORMATION_SCHEMA TABLES 表”。
