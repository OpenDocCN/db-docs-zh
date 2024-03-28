# 15.1.5 ALTER INSTANCE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-instance.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-instance.html)

```sql
ALTER INSTANCE *instance_action*

*instance_action*: {
  | {ENABLE|DISABLE} INNODB REDO_LOG
  | ROTATE INNODB MASTER KEY
  | ROTATE BINLOG MASTER KEY
  | RELOAD TLS
      [FOR CHANNEL {mysql_main | mysql_admin}]
      [NO ROLLBACK ON ERROR]
  | RELOAD KEYRING
}
```

`ALTER INSTANCE`定义适用于 MySQL 服务器实例的操作。该语句支持以下操作：

+   `ALTER INSTANCE {ENABLE | DISABLE} INNODB REDO_LOG`

    此操作启用或禁用`InnoDB`重做日志记录。默认情况下启用重做日志记录。此功能仅用于将数据加载到新的 MySQL 实例中。该语句不会写入二进制日志。此操作在 MySQL 8.0.21 中引入。

    警告

    *不要在生产系统上禁用重做日志记录。*虽然允许在禁用重做日志记录时关闭并重新启动服务器，但在禁用重做日志记录时发生意外服务器停止可能会导致数据丢失和实例损坏。

    [`ALTER INSTANCE [ENABLE|DISABLE] INNODB REDO_LOG`](alter-instance.html "15.1.5 ALTER INSTANCE 语句")操作需要独占备份锁，这会阻止其他`ALTER INSTANCE`操作同时执行。其他`ALTER INSTANCE`操作必须等待锁被释放后才能执行。

    更多信息，请参见禁用重做日志记录。

+   `ALTER INSTANCE ROTATE INNODB MASTER KEY`

    此操作旋转用于`InnoDB`表空间加密的主加密密钥。密钥旋转需要`ENCRYPTION_KEY_ADMIN`或`SUPER`权限。要执行此操作，必须安装和配置一个密钥环插件。有关说明，请参见第 8.4.4 节，“MySQL 密钥环”。

    `ALTER INSTANCE ROTATE INNODB MASTER KEY`支持并发 DML。但是，它不能与`CREATE TABLE ... ENCRYPTION`或`ALTER TABLE ... ENCRYPTION`操作同时运行，并且会获取锁以防止这些语句的并发执行可能引起的冲突。如果其中一个冲突的语句正在运行，则必须等待其完成后才能继续执行另一个。

    `ALTER INSTANCE ROTATE INNODB MASTER KEY`语句会写入二进制日志，以便在复制服务器上执行。

    有关额外的`ALTER INSTANCE ROTATE INNODB MASTER KEY`使用信息，请参见第 17.13 节，“InnoDB 数据静止加密”。

+   `ALTER INSTANCE ROTATE BINLOG MASTER KEY`

    此操作旋转用于二进制日志加密的二进制日志主密钥。二进制日志主密钥的密钥轮换需要 `BINLOG_ENCRYPTION_ADMIN` 或 `SUPER` 权限。如果 `binlog_encryption` 系统变量设置为 `OFF`，则不能使用该语句。要执行此操作，必须安装和配置一个密钥环插件。有关说明，请参阅第 8.4.4 节，“MySQL 密钥环”。

    `ALTER INSTANCE ROTATE BINLOG MASTER KEY` 操作不会写入二进制日志，也不会在副本上执行。因此，二进制日志主密钥轮换可以在包含不同 MySQL 版本的复制环境中进行。要在所有适用的源和副本服务器上安排定期轮换二进制日志主密钥，您可以在每个服务器上启用 MySQL 事件调度程序，并使用 `CREATE EVENT` 语句发出 `ALTER INSTANCE ROTATE BINLOG MASTER KEY` 语句。如果您因为怀疑当前或任何以前的二进制日志主密钥可能已被泄露而轮换二进制日志主密钥，则在每个适用的源和副本服务器上发出该语句，这样可以验证立即的合规性。

    有关其他 `ALTER INSTANCE ROTATE BINLOG MASTER KEY` 使用信息，包括如果进程未正确完成或被意外服务器停机中断时该怎么办，请参阅第 19.3.2 节，“加密二进制日志文件和中继日志文件”。

+   `ALTER INSTANCE RELOAD TLS`

    这个操作重新配置了 TLS 上下文，使用当前定义上下文的系统变量的值。它还更新了反映活动上下文值的状态变量。此操作需要`CONNECTION_ADMIN` 权限。有关重新配置 TLS 上下文的其他信息，包括哪些系统和状态变量与上下文相关，请参阅服务器端加密连接的运行时配置和监控。

    默认情况下，该语句重新加载主连接接口的 TLS 上下文。如果提供了`FOR CHANNEL`子句（自 MySQL 8.0.21 起可用），该语句将重新加载命名通道的 TLS 上下文：`mysql_main`用于主连接接口，`mysql_admin`用于管理连接接口。有关不同接口的信息，请参见第 7.1.12.1 节，“连接接口”。更新后的 TLS 上下文属性在 Performance Schema `tls_channel_status`表中公开。请参见第 29.12.21.9 节，“tls_channel_status 表”。

    更新主接口的 TLS 上下文也可能会影响管理接口，因为除非为该接口配置了一些非默认的 TLS 值，否则它将使用与主接口相同的 TLS 上下文。

    注意

    当重新加载 TLS 上下文时，OpenSSL 会重新加载包含 CRL（证书吊销列表）的文件作为过程的一部分。如果 CRL 文件很大，服务器会分配大块内存（文件大小的十倍），在加载新实例并且旧实例尚未释放时会将其加倍。大量分配被释放后，进程驻留内存不会立即减少，因此如果反复使用带有大型 CRL 文件的`ALTER INSTANCE RELOAD TLS`语句，进程驻留内存使用量可能会增加。

    默认情况下，如果配置值不允许创建新的 TLS 上下文，则`RELOAD TLS`操作会回滚并显示错误，不会产生任何效果。先前的上下文值将继续用于新连接。如果给出了可选的`NO ROLLBACK ON ERROR`子句并且无法创建新上下文，则不会发生回滚。相反，会生成警告，并且对语句适用的接口上的新连接将禁用加密。

    `ALTER INSTANCE RELOAD TLS`语句不会写入二进制日志（因此不会被复制）。TLS 配置是本地的，并且依赖于本地文件，不一定存在于所有涉及的服务器上。

+   `ALTER INSTANCE RELOAD KEYRING`

    如果安装了密钥环组件，则此操作会告诉组件重新读取其配置文件并重新初始化任何密钥环内存数据。如果您在运行时修改了组件配置，则新配置在执行此操作之前不会生效。重新加载密钥环需要`ENCRYPTION_KEY_ADMIN`权限。此操作是在 MySQL 8.0.24 中添加的。

    此操作仅允许重新配置当前安装的密钥环组件。它不允许更改已安装的组件。例如，如果您更改了已安装的密钥环组件的配置，`ALTER INSTANCE RELOAD KEYRING`会使新配置生效。另一方面，如果您更改了服务器清单文件中命名的密钥环组件，`ALTER INSTANCE RELOAD KEYRING`没有效果，当前组件仍然安装。

    `ALTER INSTANCE RELOAD KEYRING`语句不会写入二进制日志（因此不会被复制）。
