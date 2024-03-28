# 17.17.2 启用 InnoDB 监视器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-enabling-monitors.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-enabling-monitors.html)

当为定期输出启用 `InnoDB` 监视器时，`InnoDB` 每隔约 15 秒将输出写入到 **mysqld** 服务器标准错误输出 (`stderr`)。

`InnoDB` 将监视器输出发送到 `stderr` 而不是 `stdout` 或固定大小的内存缓冲区，以避免潜在的缓冲区溢出。

在 Windows 上，`stderr` 通常会被定向到默认日志文件，除非另有配置。如果想将输出定向到控制台窗口而不是错误日志，请在控制台窗口中的命令提示符下使用 `--console` 选项启动服务器。更多信息，请参见 Windows 上的默认错误日志目的地。

在 Unix 和类 Unix 系统上，`stderr` 通常会被定向到终端，除非另有配置。更多信息，请参见 Unix 和类 Unix 系统上的默认错误日志目的地。

只有在实际需要查看监视器信息时才应启用 `InnoDB` 监视器，因为输出生成会导致一定的性能降低。此外，如果监视器输出被定向到错误日志，如果忘记后续禁用监视器，日志可能会变得非常庞大。

注意

为了帮助故障排除，在某些条件下，`InnoDB` 会临时启用标准 `InnoDB` 监视器输出。更多信息，请参见 第 17.21 节，“InnoDB 故障排除”。

`InnoDB` 监视器输出以包含时间戳和监视器名称的标题开头。例如：

```sql
=====================================
2014-10-16 18:37:29 0x7fc2a95c1700 INNODB MONITOR OUTPUT
=====================================
```

标准 `InnoDB` 监视器的标题 (`INNODB MONITOR OUTPUT`) 也用于锁监视器，因为后者会生成相同的输出并附加额外的锁信息。

`innodb_status_output` 和 `innodb_status_output_locks` 系统变量用于启用标准 `InnoDB` 监视器和 `InnoDB` 锁监视器。

需要 `PROCESS` 权限才能启用或禁用 `InnoDB` 监视器。

#### 启用标准 `InnoDB` 监视器

通过将 `innodb_status_output` 系统变量设置为 `ON` 来启用标准 `InnoDB` 监视器。

```sql
SET GLOBAL innodb_status_output=ON;
```

要禁用标准 `InnoDB` 监视器，请将 `innodb_status_output` 设置为 `OFF`。

当关闭服务器时，`innodb_status_output` 变量将设置为默认值 `OFF`。

#### 启用 InnoDB 锁监视器

`InnoDB` 锁监视器数据与 `InnoDB` 标准监视器输出一起打印。必须同时启用 `InnoDB` 标准监视器和 `InnoDB` 锁监视器，才能定期打印 `InnoDB` 锁监视器数据。

要启用 `InnoDB` 锁监视器，请将 `innodb_status_output_locks` 系统变量设置为 `ON`。必须同时启用 `InnoDB` 标准监视器和 `InnoDB` 锁监视器，才能定期打印 `InnoDB` 锁监视器数据：

```sql
SET GLOBAL innodb_status_output=ON;
SET GLOBAL innodb_status_output_locks=ON;
```

要禁用 `InnoDB` 锁监视器，请将 `innodb_status_output_locks` 设置为 `OFF`。将 `innodb_status_output` 设置为 `OFF` 也会禁用 `InnoDB` 标准监视器。

当关闭服务器时，`innodb_status_output` 和 `innodb_status_output_locks` 变量将设置为默认值 `OFF`。

注意

要为 `SHOW ENGINE INNODB STATUS` 输出启用 `InnoDB` 锁监视器，只需启用 `innodb_status_output_locks` 即可。

#### 根据需要获取标准 InnoDB 监视器输出

作为定期输出标准 `InnoDB` 监视器的替代方法，您可以根据需要使用 `SHOW ENGINE INNODB STATUS` SQL 语句获取标准 `InnoDB` 监视器输出，该语句将输出提取到您的客户端程序。如果您使用 **mysql** 交互式客户端，则如果将通常的分号语句终止符替换为 `\G`，输出将更易读：

```sql
mysql> SHOW ENGINE INNODB STATUS\G
```

`SHOW ENGINE INNODB STATUS` 输出还包括 `InnoDB` 锁监视器数据，如果启用了 `InnoDB` 锁监视器。

#### 将标准 InnoDB 监视器输出定向到状态文件

可以通过在启动时指定 `--innodb-status-file` 选项来启用并将标准 `InnoDB` 监视器输出定向到状态文件。使用此选项时，`InnoDB` 在数据目录中创建一个名为 `innodb_status.*`pid`*` 的文件，并每隔约 15 秒将输出写入其中。

当服务器正常关闭时，`InnoDB` 会删除状态文件。如果发生异常关闭，则可能需要手动删除状态文件。

`--innodb-status-file` 选项仅供临时使用，因为输出生成可能会影响性能，并且随着时间推移，`innodb_status.*`pid`*` 文件可能会变得非常大。
