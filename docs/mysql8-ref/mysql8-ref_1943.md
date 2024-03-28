# 28.3.48 The INFORMATION_SCHEMA VIEWS Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-views-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-views-table.html)

`VIEWS` 表提供有关数据库中视图的信息。您必须具有 `SHOW VIEW` 权限才能访问此表。

`VIEWS` 表包含以下列：

+   `TABLE_CATALOG`

    视图所属目录的名称。该值始终为 `def`。

+   `TABLE_SCHEMA`

    视图所属模式（数据库）的名称。

+   `TABLE_NAME`

    视图的名称。

+   `VIEW_DEFINITION`

    提供视图定义的 `SELECT` 语句。该列包含 `SHOW CREATE VIEW` 生成的 `Create Table` 列中的大部分内容。跳过 `SELECT` 前的单词和跳过 `WITH CHECK OPTION` 前的单词。假设原始语句为：

    ```sql
    CREATE VIEW v AS
      SELECT s2,s1 FROM t
      WHERE s1 > 5
      ORDER BY s1
      WITH CHECK OPTION;
    ```

    然后视图定义如下：

    ```sql
    SELECT s2,s1 FROM t WHERE s1 > 5 ORDER BY s1
    ```

+   `CHECK_OPTION`

    `CHECK_OPTION` 属性的值。该值为 `NONE`、`CASCADE` 或 `LOCAL` 中的一个。

+   `IS_UPDATABLE`

    MySQL 在 `CREATE VIEW` 时设置一个标志，称为视图可更新性标志。如果视图可以进行 `UPDATE` 和 `DELETE`（以及类似操作），则该标志设置为 `YES`（true）。否则，该标志设置为 `NO`（false）。`VIEWS` 表中的 `IS_UPDATABLE` 列显示了该标志的状态。这意味着服务器始终知道视图是否可更新。

    如果视图不可更新，则诸如 `UPDATE`、`DELETE` 和 `INSERT` 等语句是非法的并将被拒绝。（即使视图是可更新的，也可能无法向其插入数据；有关详细信息，请参阅 Section 27.5.3, “Updatable and Insertable Views”。）

+   `DEFINER`

    创建视图的用户帐户，格式为 `'*`user_name`*'@'*`host_name`*'`。

+   `SECURITY_TYPE`

    视图的 `SQL SECURITY` 特性。该值为 `DEFINER` 或 `INVOKER` 中的一个。

+   `CHARACTER_SET_CLIENT`

    视图创建时 `character_set_client` 系统变量的会话值。

+   `COLLATION_CONNECTION`

    视图创建时 `collation_connection` 系统变量的会话值。

#### 注意事项

MySQL 允许不同的`sql_mode`设置告诉服务器支持的 SQL 语法类型。例如，您可以使用`ANSI` SQL 模式来确保 MySQL 正确解释标准 SQL 连接运算符，双竖线(`||`)，在您的查询中。如果您创建一个连接项目的视图，您可能担心将`sql_mode`设置更改为与`ANSI`不同的值会导致视图无效。但事实并非如此。无论您如何编写视图定义，MySQL 始终以相同的方式存储它，即规范形式。以下是一个示例，显示服务器如何将双竖线连接运算符更改为`CONCAT()`函数：

```sql
mysql> SET sql_mode = 'ANSI';
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE VIEW test.v AS SELECT 'a' || 'b' as col1;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT VIEW_DEFINITION FROM INFORMATION_SCHEMA.VIEWS
       WHERE TABLE_SCHEMA = 'test' AND TABLE_NAME = 'v';
+----------------------------------+
| VIEW_DEFINITION                  |
+----------------------------------+
| select concat('a','b') AS `col1` |
+----------------------------------+
1 row in set (0.00 sec)
```

将视图定义存储为规范形式的优势在于，稍后对`sql_mode`值的更改不会影响视图的结果。然而，另一个结果是，服务器会剥离`SELECT`之前的注释。
