> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-current-user.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-current-user.html)

#### 19.5.1.8 CURRENT_USER() 的复制

以下语句支持使用`CURRENT_USER()`函数来代替受影响用户或定义者的名称，以及可能的主机：

+   `DROP USER`

+   `RENAME USER`

+   `GRANT`

+   `REVOKE`

+   `CREATE FUNCTION`

+   `CREATE PROCEDURE`

+   `CREATE TRIGGER`

+   `CREATE EVENT`

+   `CREATE VIEW`

+   `ALTER EVENT`

+   `ALTER VIEW`

+   `SET PASSWORD`

当启用二进制日志记录并且在这些语句中使用`CURRENT_USER()`或`CURRENT_USER`作为定义者时，MySQL 服务器确保在语句被复制时应用于源和副本上的相同用户。在某些情况下，例如更改密码的语句，函数引用在写入二进制日志之前会被展开，以便语句包含用户名称。对于所有其他情况，源上的当前用户名称会作为元数据被复制到副本上，并且副本会将语句应用于元数据中命名的当前用户，而不是副本上的当前用户。
