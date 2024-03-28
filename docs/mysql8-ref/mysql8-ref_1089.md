> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-create-view.html`](https://dev.mysql.com/doc/refman/8.0/en/show-create-view.html)

#### 15.7.7.13 显示创建视图语句

```sql
SHOW CREATE VIEW *view_name*
```

此语句显示创建命名视图的`CREATE VIEW`语句。

```sql
mysql> SHOW CREATE VIEW v\G
*************************** 1\. row ***************************
                View: v
         Create View: CREATE ALGORITHM=UNDEFINED
                      DEFINER=`bob`@`localhost`
                      SQL SECURITY DEFINER VIEW
                      `v` AS select 1 AS `a`,2 AS `b`
character_set_client: utf8mb4
collation_connection: utf8mb4_0900_ai_ci
```

`character_set_client` 是视图创建时`character_set_client`系统变量的会话值。`collation_connection` 是视图创建时`collation_connection`系统变量的会话值。

使用`SHOW CREATE VIEW`需要`SHOW VIEW`权限，并且需要针对特定视图的`SELECT`权限。

视图信息也可以从`INFORMATION_SCHEMA` `VIEWS`表中获取。参见 Section 28.3.48, “INFORMATION_SCHEMA VIEWS 表”。

MySQL 允许您使用不同的`sql_mode`设置来告诉服务器支持的 SQL 语法类型。例如，您可以使用`ANSI` SQL 模式来确保 MySQL 正确解释标准 SQL 连接运算符，双竖线(`||`)，在您的查询中。如果您创建一个连接项目的视图，您可能担心将`sql_mode`设置更改为与`ANSI`不同的值会导致视图无效。但事实并非如此。无论您如何编写视图定义，MySQL 始终以规范形式存储它。以下是一个示例，显示服务器如何将双竖线连接运算符更改为`CONCAT()`函数：

```sql
mysql> SET sql_mode = 'ANSI';
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE VIEW test.v AS SELECT 'a' || 'b' as col1;
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW CREATE VIEW test.v\G
*************************** 1\. row ***************************
                View: v
         Create View: CREATE VIEW "v" AS select concat('a','b') AS "col1"
... 1 row in set (0.00 sec)
```

将视图定义存储为规范形式的优势在于稍后对`sql_mode`值的更改不会影响视图的结果。然而，另一个后果是服务器会剥离`SELECT`之前的注释。
