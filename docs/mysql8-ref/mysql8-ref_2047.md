> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-session-account-connect-attrs-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-session-account-connect-attrs-table.html)

#### 29.12.9.1 session_account_connect_attrs 表

应用程序可以在连接时提供键值连接属性以传递给服务器。有关常见属性的描述，请参见 Section 29.12.9, “Performance Schema Connection Attribute Tables”。

`session_account_connect_attrs` 表仅包含当前会话的连接属性，以及与会话账户关联的其他会话。要查看所有会话的连接属性，请使用 `session_connect_attrs` 表。

`session_account_connect_attrs` 表包含以下列：

+   `PROCESSLIST_ID`

    会话的连接标识符。

+   `ATTR_NAME`

    属性名称。

+   `ATTR_VALUE`

    属性值。

+   `ORDINAL_POSITION`

    属性添加到连接属性集的顺序。

`session_account_connect_attrs` 表包含以下索引：

+   主键为 (`PROCESSLIST_ID`, `ATTR_NAME`)

`TRUNCATE TABLE` 不允许用于 `session_account_connect_attrs` 表。
