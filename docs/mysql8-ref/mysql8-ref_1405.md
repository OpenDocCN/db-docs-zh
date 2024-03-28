> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-privilege-checks-gr.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-privilege-checks-gr.html)

#### 19.3.3.2 组复制通道的权限检查

从 MySQL 8.0.19 开始，除了保护异步和半同步复制外，您还可以选择使用`PRIVILEGE_CHECKS_USER`帐户来保护组复制使用的两个复制应用程序线程。每个组成员上的`group_replication_applier`线程用于应用组的事务，每个组成员上的`group_replication_recovery`线程用于从二进制日志中进行状态传输，作为成员加入或重新加入组时的分布式恢复的一部分。

要保护其中一个线程，停止组复制，然后使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）并指定`PRIVILEGE_CHECKS_USER`选项，将`group_replication_applier`或`group_replication_recovery`作为通道名称。例如：

```sql
mysql> STOP GROUP_REPLICATION;
mysql> CHANGE MASTER TO PRIVILEGE_CHECKS_USER = 'gr_repl'@'%.example.com' 
          FOR CHANNEL 'group_replication_recovery';
mysql> FLUSH PRIVILEGES;
mysql> START GROUP_REPLICATION;

Or from MySQL 8.0.23:
mysql> STOP GROUP_REPLICATION;
mysql> CHANGE REPLICATION SOURCE TO PRIVILEGE_CHECKS_USER = 'gr_repl'@'%.example.com' 
          FOR CHANNEL 'group_replication_recovery';
mysql> FLUSH PRIVILEGES;
mysql> START GROUP_REPLICATION;
```

对于组复制通道，当通道创建时会自动启用`REQUIRE_ROW_FORMAT`设置，无法禁用，因此您无需指定此设置。

重要提示

在 MySQL 8.0.19 中，请确保在运行组复制时不要使用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句与`PRIVILEGE_CHECKS_USER`选项。此操作会导致通道的中继日志文件被清除，可能导致已接收并排队在中继日志中但尚未应用的事务丢失。

Group Replication 要求每个要被组复制的表都必须有一个定义的主键，或者等效的主键，其中等效的主键是一个非空唯一键。与`sql_require_primary_key`系统变量执行的检查不同，Group Replication 有自己内置的一套用于主键或主键等效的检查。您可以为 Group Replication 通道设置`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句的`REQUIRE_TABLE_PRIMARY_KEY_CHECK`选项为`ON`。但是，请注意，您可能会发现一些在 Group Replication 内置检查下允许的事务，在设置`sql_require_primary_key = ON`或`REQUIRE_TABLE_PRIMARY_KEY_CHECK = ON`时不被允许。因此，从 MySQL 8.0.20（引入该选项时）开始，新的和升级的 Group Replication 通道将`REQUIRE_TABLE_PRIMARY_KEY_CHECK`设置为默认的`STREAM`，而不是`ON`。

如果在 Group Replication 中使用远程克隆操作进行分布式恢复（参见 Section 20.5.4.2, “Cloning for Distributed Recovery”），从 MySQL 8.0.19 开始，`PRIVILEGE_CHECKS_USER`账户和相关设置从捐赠方克隆到加入成员。如果设置加入成员在启动时启动 Group Replication，则它会自动使用该账户对适当的复制通道进行特权检查。

在 MySQL 8.0.18 中，由于一些限制，建议不要在 Group Replication 通道中使用`PRIVILEGE_CHECKS_USER`账户。
