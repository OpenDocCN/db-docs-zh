> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-session-connect-attrs-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-session-connect-attrs-table.html)

#### 29.12.9.2 session_connect_attrs 表

应用程序可以在连接时提供键值连接属性，以传递给服务器。有关常见属性的描述，请参见第 29.12.9 节，“性能模式连接属性表”。

`session_connect_attrs` 表包含所有会话的连接属性。要仅查看当前会话的连接属性以及与会话帐户关联的其他会话，请使用`session_account_connect_attrs` 表。

`session_connect_attrs` 表包含以下列：

+   `PROCESSLIST_ID`

    会话的连接标识符。

+   `ATTR_NAME`

    属性名称。

+   `ATTR_VALUE`

    属性值。

+   `ORDINAL_POSITION`

    属性添加到连接属性集的顺序。

`session_connect_attrs` 表包含以下索引：

+   主键 (`PROCESSLIST_ID`, `ATTR_NAME`)

不允许对`session_connect_attrs` 表执行`TRUNCATE TABLE`。
