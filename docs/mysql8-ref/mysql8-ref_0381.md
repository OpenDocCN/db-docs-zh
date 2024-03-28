# 8.2.21 设置帐户资源限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/user-resources.html`](https://dev.mysql.com/doc/refman/8.0/en/user-resources.html)

限制客户端使用 MySQL 服务器资源的一种方法是将全局`max_user_connections`系统变量设置为非零值。这限制了任何给定帐户可以建立的同时连接数，但不限制客户端连接后可以做的事情。此外，设置`max_user_connections`不会启用对个别帐户的管理。这两种类型的控制对 MySQL 管理员很重要。

为了解决这些问题，MySQL 允许对个别帐户使用这些服务器资源的限制：

+   一个帐户每小时可以发出的查询次数

+   一个帐户每小时可以发出的更新次数

+   一个帐户每小时可以连接到服务器的次数

+   一个帐户对服务器的同时连接数

客户端可以发出的任何语句都计入查询限制。只有修改数据库或表的语句才计入更新限制。

在这种情况下，“帐户”对应于`mysql.user`系统表中的一行。也就是说，连接根据适用于连接的`user`表行中的`User`和`Host`值进行评估。例如，帐户`'usera'@'%.example.com'`对应于`user`表中具有`User`和`Host`值为`usera`和`%.example.com`的行，以允许`usera`从`example.com`域中的任何主机连接。在这种情况下，服务器将在此行中的资源限制集体应用于`usera`从`example.com`域中的任何主机发起的所有连接，因为所有这样的连接使用相同的帐户。

在 MySQL 5.0 之前，“帐户”是根据用户连接的实际主机进行评估的。可以通过使用`--old-style-user-limits`选项启动服务器来选择这种较旧的计算方法。在这种情况下，如果`usera`同时从`host1.example.com`和`host2.example.com`连接，服务器会将帐户资源限制分别应用于每个连接。如果`usera`再次从`host1.example.com`连接，服务器将与该主机的现有连接一起应用该连接的限制。

注意

在 MySQL 8.0.30 中，`--old-style-user-limits`选项已被弃用，并可能在将来的 MySQL 版本中被移除。在 MySQL 8.0.30 或更高版本中在命令行或选项文件中使用此选项会导致服务器发出警告。

要在创建帐户时为帐户建立资源限制，请使用`CREATE USER`语句。要修改现有帐户的限制，请使用`ALTER USER`。提供一个`WITH`子句，命名要限制的每个资源。每个限制的默认值为零（无限制）。例如，要创建一个新帐户，可以访问`customer`数据库，但只能以有限的方式访问，发出以下语句：

```sql
mysql> CREATE USER 'francis'@'localhost' IDENTIFIED BY 'frank'
 ->     WITH MAX_QUERIES_PER_HOUR 20
 ->          MAX_UPDATES_PER_HOUR 10
 ->          MAX_CONNECTIONS_PER_HOUR 5
 ->          MAX_USER_CONNECTIONS 2;
```

限制类型不需要全部在`WITH`子句中命名，但命名的限制可以以任何顺序出现。每小时限制的值应为表示每小时计数的整数。对于`MAX_USER_CONNECTIONS`，限制是表示帐户同时连接数的整数。如果将此限制设置为零，则全局`max_user_connections`系统变量值确定同时连接数。如果`max_user_connections`也为零，则该帐户没有限制。

要修改现有帐户的限制，请使用`ALTER USER`。以下语句将`francis`的查询限制更改为 100：

```sql
mysql> ALTER USER 'francis'@'localhost' WITH MAX_QUERIES_PER_HOUR 100;
```

该语句仅修改指定的限制值，不会对帐户进行其他更改。

要删除限制，请将其值设置为零。例如，要删除`francis`每小时连接次数限制，使用以下语句：

```sql
mysql> ALTER USER 'francis'@'localhost' WITH MAX_CONNECTIONS_PER_HOUR 0;
```

如前所述，帐户的同时连接限制是根据`MAX_USER_CONNECTIONS`限制和`max_user_connections`系统变量确定的。假设全局`max_user_connections`值为 10，并且有三个帐户具有以下指定的各自资源限制：

```sql
ALTER USER 'user1'@'localhost' WITH MAX_USER_CONNECTIONS 0;
ALTER USER 'user2'@'localhost' WITH MAX_USER_CONNECTIONS 5;
ALTER USER 'user3'@'localhost' WITH MAX_USER_CONNECTIONS 20;
```

`user1`的连接限制为 10（全局`max_user_connections`值），因为它的`MAX_USER_CONNECTIONS`限制为零。`user2`和`user3`的连接限制分别为 5 和 20，因为它们具有非零的`MAX_USER_CONNECTIONS`限制。

服务器将帐户的资源限制存储在与帐户对应的`user`表行中。`max_questions`、`max_updates`和`max_connections`列存储每小时限制，`max_user_connections`列存储`MAX_USER_CONNECTIONS`限制。（参见第 8.2.3 节，“授权表”。）

当任何帐户对其任何资源的使用放置了非零限制时，资源使用计数会发生。

当服务器运行时，它会计算每个账户使用资源的次数。如果一个账户在最近一小时内达到了连接数限制，服务器将拒绝该账户的进一步连接，直到该小时结束。同样，如果账户达到了查询或更新次数限制，服务器将拒绝进一步的查询或更新，直到该小时结束。在所有这些情况下，服务器会发出适当的错误消息。

资源计数是按账户而不是按客户端进行的。例如，如果您的账户有一个查询限制为 50，您不能通过同时在服务器上建立两个客户端连接来将限制增加到 100。两个连接上发出的查询会被一起计算。

当前每小时资源使用计数可以全局重置所有账户，或者针对特定账户进行单独重置：

+   要将当前计数重置为零以适用于所有账户，请发出一个`FLUSH USER_RESOURCES`语句。也可以通过重新加载授权表（例如，使用`FLUSH PRIVILEGES`语句或**mysqladmin reload**命令）来重置计数。

+   可以通过再次设置任何限制值将单个账户的计数重置为零。指定一个与当前分配给账户的值相等的限制值。

每小时计数重置不会影响`MAX_USER_CONNECTIONS`限制。

所有计数在服务器启动时都从零开始。计数不会在服务器重新启动时保留。

对于`MAX_USER_CONNECTIONS`限制，如果账户当前已经打开了允许的最大连接数，则可能出现一个边界情况：快速断开连接然后立即重新连接可能会导致错误（`ER_TOO_MANY_USER_CONNECTIONS`或`ER_USER_LIMIT_REACHED`），如果服务器在重新连接发生时还没有完全处理断开连接。当服务器完成断开连接处理后，又可以再次允许另一个连接。
