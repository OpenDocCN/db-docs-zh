# 8.2.14 分配帐户密码

> 原文：[`dev.mysql.com/doc/refman/8.0/en/assigning-passwords.html`](https://dev.mysql.com/doc/refman/8.0/en/assigning-passwords.html)

连接到 MySQL 服务器的客户端所需的凭据可能包括密码。本节描述了如何为 MySQL 帐户分配密码。

MySQL 将凭据存储在`mysql`系统数据库中的`user`表中。仅允许具有`CREATE USER`权限的用户执行分配或修改密码的操作，或者具有`mysql`数据库的权限（`INSERT`权限用于创建新帐户，`UPDATE`权限用于修改现有帐户）。如果启用了`read_only`系统变量，则使用诸如`CREATE USER`或`ALTER USER`之类的帐户修改语句还需要`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）。

这里的讨论仅总结了最常见的密码分配语句的语法。有关其他可能性的完整详细信息，请参见第 15.7.1.3 节，“CREATE USER 语句”，第 15.7.1.1 节，“ALTER USER 语句”和第 15.7.1.10 节，“SET PASSWORD 语句”。

MySQL 使用插件执行客户端身份验证；请参见第 8.2.17 节，“可插拔身份验证”。在分配密码的语句中，与帐户关联的身份验证插件执行指定的明文密码所需的任何哈希处理。这使得 MySQL 能够在将密码存储在`mysql.user`系统表中之前对其进行混淆。对于这里描述的语句，MySQL 会自动对指定的密码进行哈希处理。还有用于`CREATE USER`和`ALTER USER`的语法，允许直接指定哈希值。有关详细信息，请参阅这些语句的描述。

在创建新帐户时分配密码，请使用`CREATE USER`并包含`IDENTIFIED BY`子句：

```sql
CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY '*password*';
```

`CREATE USER`还支持指定帐户身份验证插件的语法。请参见第 15.7.1.3 节，“CREATE USER 语句”。

要为现有帐户分配或更改密码，请使用带有`IDENTIFIED BY`子句的`ALTER USER`语句：

```sql
ALTER USER 'jeffrey'@'localhost' IDENTIFIED BY '*password*';
```

如果您未连接为匿名用户，可以在不明确命名自己的帐户的情况下更改自己的密码：

```sql
ALTER USER USER() IDENTIFIED BY '*password*';
```

要从命令行更改帐户密码，请使用**mysqladmin**命令：

```sql
mysqladmin -u *user_name* -h *host_name* password "*password*"
```

此命令设置密码的帐户是`mysql.user`系统表中具有与`User`列中的*`user_name`*和`Host`列中的*从中连接的客户端主机*匹配的行的帐户。

警告

使用**mysqladmin**设置密码应被视为*不安全*。在某些系统上，您的密码会对系统状态程序（如**ps**）可见，其他用户可以调用这些程序来显示命令行。MySQL 客户端通常在初始化序列期间用零覆盖命令行密码参数。然而，在这个值可见的瞬间仍然存在。此外，在某些系统上，这种覆盖策略是无效的，密码仍然对**ps**可见。（SystemV Unix 系统和其他系统可能存在这个问题。）

如果您正在使用 MySQL 复制，请注意，目前，作为`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）的一部分使用的密码实际上被限制为 32 个字符的长度；如果密码更长，任何多余的字符都将被截断。这不是由 MySQL Server 普遍强加的任何限制，而是 MySQL 复制特定的问题。
