> 译文：[`dev.mysql.com/doc/refman/8.0/en/channels-startup-options.html`](https://dev.mysql.com/doc/refman/8.0/en/channels-startup-options.html)

#### 19.2.2.3 启动选项和复制通道

本节描述了受复制通道添加影响的启动选项。

当使用复制通道时，`master_info_repository` 和 `relay_log_info_repository` 系统变量必须设置为`FILE`。在 MySQL 8.0 中，`FILE` 设置已被弃用，`TABLE` 是默认设置，因此可以省略这些系统变量。从 MySQL 8.0.23 开始，它们必须被省略，因为从该版本开始已弃用它们的使用。如果这些系统变量设置为`FILE`，则尝试向副本添加更多源将导致`ER_SLAVE_NEW_CHANNEL_WRONG_REPOSITORY`错误。

下列启动选项现在影响*所有*复制拓扑中的通道。

+   `--log-replica-updates` 或 `--log-slave-updates`

    所有副本接收的事务（甚至来自多个源）都会写入二进制日志。

+   `--relay-log-purge`

    设置后，每个通道会自动清除自己的中继日志。

+   `--replica-transaction-retries` 或 `--slave-transaction-retries`

    指定的事务重试次数可以在所有通道的所有应用程序线程上进行。

+   `--skip-replica-start` 或 `--skip-slave-start`（或 `skip_replica_start` 或 `skip_slave_start` 系统变量设置）

    任何通道上都不会启动复制线程。

+   `--replica-skip-errors` 或 `--slave-skip-errors`

    执行继续，所有通道的错误都被跳过。

下列启动选项的设置适用于每个通道；由于这些是**mysqld**启动选项，它们将应用于每个通道。

+   `--max-relay-log-size=*`size`*`

    每个通道的单个中继日志文件的最大大小；达到此限制后，文件将被轮换。

+   `--relay-log-space-limit=*`size`*`

    每个单独通道所有中继日志总大小的上限。对于*`N`*个通道，这些日志的总大小限制为`relay_log_space_limit * *`N`*`。

+   `--replica-parallel-workers=*`value`*`或`--slave-parallel-workers=*`value`*`

    每个通道的复制应用程序线程数。

+   `replica_checkpoint_group`或`slave_checkpoint_group`

    每个接收线程等待每个源的时间。

+   `--relay-log-index=filename`

    每个通道中继日志索引文件的基本名称。参见第 19.2.2.4 节，“复制通道命名约定”。

+   `--relay-log=filename`

    表示每个通道中继日志文件的基本名称。参见第 19.2.2.4 节，“复制通道命名约定”。

+   `--replica-net-timeout=N`或`--slave-net-timeout=N`

    这个值是针对每个通道设置的，因此每个通道等待*`N`*秒来检查是否存在断开的连接。

+   `--replica-skip-counter=N`或`--slave-skip-counter=N`

    这个值是针对每个通道设置的，因此每个通道跳过*`N`*个来自其源的事件。
