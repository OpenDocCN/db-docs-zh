# 19.3.3 复制权限检查

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-privilege-checks.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-privilege-checks.html)

19.3.3.1 用于复制 PRIVILEGE_CHECKS_USER 帐户的权限

19.3.3.2 Group Replication Channels 的权限检查

19.3.3.3 从失败的复制权限检查中恢复

默认情况下，MySQL 复制（包括 Group Replication）在将已被另一台服务器接受的事务应用于副本或组成员时不执行权限检查。从 MySQL 8.0.18 开始，您可以创建一个具有适当权限的用户帐户来应用通常在通道上复制的事务，并将其指定为复制应用程序的 `PRIVILEGE_CHECKS_USER` 帐户，使用 `CHANGE REPLICATION SOURCE TO` 语句（从 MySQL 8.0.23 开始）或 `CHANGE MASTER TO` 语句（在 MySQL 8.0.23 之前）。然后，MySQL 检查每个事务是否符合用户帐户的权限，以验证您已授权该通道的操作。该帐户还可以安全地由管理员使用，以应用或重新应用来自 **mysqlbinlog** 输出的事务，例如从通道上的复制错误中恢复。

使用 `PRIVILEGE_CHECKS_USER` 帐户有助于保护复制通道免受未经授权或意外使用特权或不需要的操作。在以下情况下，`PRIVILEGE_CHECKS_USER` 帐户提供了额外的安全层：

+   您正在在组织网络上的服务器实例和另一个网络上的服务器实例之间进行复制，例如云服务提供商提供的实例。

+   您希望将多个本地部署或外部部署作为单独的单元进行管理，而不是给予一个管理员帐户对所有部署的权限。

+   您希望拥有一个管理员帐户，使管理员仅能执行与复制通道和其复制的数据库直接相关的操作，而不是在服务器实例上拥有广泛的权限。

当您为通道指定 `PRIVILEGE_CHECKS_USER` 帐户时，您可以通过在 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 语句中添加以下选项之一或两者来增加应用权限检查的复制通道的安全性：

+   `REQUIRE_ROW_FORMAT`选项（从 MySQL 8.0.19 开始提供）使复制通道仅接受基于行的复制事件。当设置`REQUIRE_ROW_FORMAT`时，您必须在源服务器上使用基于行的二进制日志记录（`binlog_format=ROW`）。在 MySQL 8.0.18 中，`REQUIRE_ROW_FORMAT`不可用，但仍强烈建议在受保护的复制通道中使用基于行的二进制日志记录。使用基于语句的二进制日志记录时，可能需要一些管理员级别的特权才能使`PRIVILEGE_CHECKS_USER`账户成功执行事务。

+   `REQUIRE_TABLE_PRIMARY_KEY_CHECK`选项（从 MySQL 8.0.20 开始提供）使复制通道使用自己的主键检查策略。设置为`ON`表示始终需要主键，设置为`OFF`表示永远不需要主键。默认设置为`STREAM`，使用从源复制的每个事务的值设置`sql_require_primary_key`系统变量的会话值。当设置`PRIVILEGE_CHECKS_USER`时，将`REQUIRE_TABLE_PRIMARY_KEY_CHECK`设置为`ON`或`OFF`意味着用户帐户无需会话管理级别特权来设置受限制的会话变量，这些变量是更改`sql_require_primary_key`值所需的。它还规范化了不同源的复制通道的行为。

您授予`REPLICATION_APPLIER`权限以使用户账户能够出现为复制应用程序线程的`PRIVILEGE_CHECKS_USER`，并执行 mysqlbinlog 使用的内部`BINLOG`语句。`PRIVILEGE_CHECKS_USER`账户的用户名和主机名必须遵循第 8.2.4 节，“指定帐户名称”中描述的语法，并且用户不能是匿名用户（具有空白用户名）或`CURRENT_USER`。要创建新帐户，请使用`CREATE USER`语句。要授予此帐户`REPLICATION_APPLIER`权限，请使用`GRANT`语句。例如，要创建一个名为`priv_repl`的用户帐户，该帐户可以由`example.com`域中的任何主机的管理员手动使用，并且需要加密连接，请执行以下语句：

```sql
mysql> SET sql_log_bin = 0;
mysql> CREATE USER 'priv_repl'@'%.example.com' IDENTIFIED BY '*password*' REQUIRE SSL;
mysql> GRANT REPLICATION_APPLIER ON *.* TO 'priv_repl'@'%.example.com';
mysql> SET sql_log_bin = 1;
```

`SET sql_log_bin`语句用于使账户管理语句不会被添加到二进制日志中并发送到复制通道（参见第 15.4.1.3 节，“SET sql_log_bin 语句”）。

重要提示

`caching_sha2_password`身份验证插件是从 MySQL 8.0 开始新创建用户的默认插件（有关详细信息，请参见 Section 8.4.1.2, “Caching SHA-2 Pluggable Authentication”）。要使用使用此插件进行身份验证的用户帐户连接到服务器，您必须设置加密连接，如 Section 19.3.1, “Setting Up Replication to Use Encrypted Connections”中所述，或启用不加密连接以支持使用 RSA 密钥对进行密码交换。

设置用户帐户后，使用`GRANT`语句授予附加权限，以使用户帐户能够执行您期望应用程序线程执行的数据库更改，例如更新服务器上保存的特定表。这些相同的权限使管理员能够在需要手动执行任何这些事务的情况下使用该帐户在复制通道上执行操作。如果尝试执行未授予适当权限的意外操作，则该操作将被拒绝，并且复制应用程序线程将停止并显示错误。Section 19.3.3.1, “Privileges For The Replication PRIVILEGE_CHECKS_USER Account”解释了帐户需要的其他权限。例如，要授予`priv_repl`用户帐户向`db1`中的`cust`表添加行的`INSERT`权限，发出以下语句：

```sql
mysql> GRANT INSERT ON db1.cust TO 'priv_repl'@'%.example.com';
```

通过`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（MySQL 8.0.23 之前）为复制通道分配`PRIVILEGE_CHECKS_USER`帐户。如果复制正在运行，请在`CHANGE MASTER TO`语句之前发出`STOP REPLICA`（或在 MySQL 8.0.22 之前，`STOP SLAVE`）命令，并在其后发出`START REPLICA`。强烈建议在设置`PRIVILEGE_CHECKS_USER`时使用基于行的二进制日志记录，并且从 MySQL 8.0.19 开始，您可以使用该语句设置`REQUIRE_ROW_FORMAT`以强制执行此操作。

当重新启动复制通道时，动态特权的检查从那时开始应用。但是，静态全局特权在应用程序的上下文中不活动，直到重新加载授权表，因为这些特权不会更改为连接的客户端。要激活静态特权，请执行刷新特权操作。可以通过发出`FLUSH PRIVILEGES`语句或执行**mysqladmin flush-privileges**或**mysqladmin reload**命令来完成此操作。

例如，在运行中的 MySQL 8.0.23 及更高版本中，要在通道`channel_1`上启动特权检查，请执行以下语句：

```sql
mysql> STOP REPLICA FOR CHANNEL 'channel_1';
mysql> CHANGE REPLICATION SOURCE TO
     >    PRIVILEGE_CHECKS_USER = 'priv_repl'@'%.example.com',
     >    REQUIRE_ROW_FORMAT = 1 FOR CHANNEL 'channel_1';
mysql> FLUSH PRIVILEGES;
mysql> START REPLICA FOR CHANNEL 'channel_1';
```

在 MySQL 8.0.23 之前，您可以使用以下显示的语句：

```sql
mysql> STOP SLAVE FOR CHANNEL 'channel_1';
mysql> CHANGE MASTER TO
     >    PRIVILEGE_CHECKS_USER = 'priv_repl'@'%.example.com',
     >    REQUIRE_ROW_FORMAT = 1 FOR CHANNEL 'channel_1';
mysql> FLUSH PRIVILEGES;
mysql> START SLAVE FOR CHANNEL 'channel_1';
```

如果未指定通道且不存在其他通道，则该语句将应用于默认通道。在性能模式`replication_applier_configuration`表中显示了频道的`PRIVILEGE_CHECKS_USER`帐户的用户名和主机名，它们已经适当转义，因此可以直接复制到 SQL 语句中以执行单个事务。

在 MySQL 8.0.31 及更高版本中，如果使用`Rewriter`插件，则应授予`PRIVILEGE_CHECKS_USER`用户帐户`SKIP_QUERY_REWRITE`特权。这可以防止此用户发出的语句被重写。有关更多信息，请参见第 7.6.4 节，“Rewriter Query Rewrite Plugin”。

当为复制通道设置`REQUIRE_ROW_FORMAT`时，复制应用程序不会创建或删除临时表，因此不会设置`pseudo_thread_id`会话系统变量。它不执行`LOAD DATA INFILE`指令，因此不会尝试文件操作以访问或删除与数据加载相关的临时文件（作为`Format_description_log_event`记录）。它不执行`INTVAR`、`RAND`和`USER_VAR`事件，这些事件用于重现基于语句的复制的客户端连接状态。 （一个例外是与 DDL 查询相关的`USER_VAR`事件，这些事件将被执行。）它不执行在 DML 事务中记录的任何语句。如果复制应用程序在尝试排队或应用事务时检测到这些类型的事件之一，则不会应用该事件，并且复制将因错误而停止。

无论是否设置了`PRIVILEGE_CHECKS_USER`账户，都可以为复制通道设置`REQUIRE_ROW_FORMAT`。即使没有特权检查，当设置此选项时实施的限制也会增加复制通道的安全性。在使用**mysqlbinlog**时，还可以指定`--require-row-format`选项，以强制在**mysqlbinlog**输出中执行基于行的复制事件。

**安全上下文。** 默认情况下，当使用指定的用户账户启动复制应用程序线程作为`PRIVILEGE_CHECKS_USER`时，安全上下文将使用默认角色创建，或者如果`activate_all_roles_on_login`设置为`ON`，则使用所有角色。

你可以使用角色为作为`PRIVILEGE_CHECKS_USER`账户使用的账户提供一般的权限集，就像以下示例中所示。在这里，与之前的示例中直接向用户账户授予`db1.cust`表的`INSERT`权限不同，这个权限被授予给了角色`priv_repl_role`，同时还有`REPLICATION_APPLIER`权限。然后使用该角色将权限集授予两个用户账户，这两个账户现在都可以作为`PRIVILEGE_CHECKS_USER`账户使用：

```sql
mysql> SET sql_log_bin = 0;
mysql> CREATE USER 'priv_repa'@'%.example.com'
                  IDENTIFIED BY '*password*'
                  REQUIRE SSL;
mysql> CREATE USER 'priv_repb'@'%.example.com'
                  IDENTIFIED BY '*password*'
                  REQUIRE SSL;
mysql> CREATE ROLE 'priv_repl_role';
mysql> GRANT REPLICATION_APPLIER TO 'priv_repl_role';
mysql> GRANT INSERT ON db1.cust TO 'priv_repl_role';
mysql> GRANT 'priv_repl_role' TO
                  'priv_repa'@'%.example.com',
                  'priv_repb'@'%.example.com';
mysql> SET DEFAULT ROLE 'priv_repl_role' TO
                  'priv_repa'@'%.example.com',
                  'priv_repb'@'%.example.com';
mysql> SET sql_log_bin = 1;
```

请注意，当复制应用程序线程创建安全上下文时，它会检查`PRIVILEGE_CHECKS_USER`账户的权限，但不执行密码验证，并且不执行与账户管理相关的检查，例如检查账户是否被锁定。创建的安全上下文在复制应用程序线程的生命周期内保持不变。

**限制。** 在 MySQL 8.0.18 版本中，如果在发出`RESET REPLICA`语句后立即重新启动复制**mysqld**（由于意外服务器退出或有意重启），则`mysql.slave_relay_log_info`表中保存的`PRIVILEGE_CHECKS_USER`账户设置将丢失，必须重新指定。在使用特权检查时，始终在重新启动后验证它们是否存在，并在需要时重新指定。从 MySQL 8.0.19 开始，在这种情况下，`PRIVILEGE_CHECKS_USER`账户设置将被保留，因此它将从表中检索并重新应用到通道中。
