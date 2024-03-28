# 14.18.4 基于位置的同步函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-functions-synchronization.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-functions-synchronization.html)

此部分列出的函数用于控制 MySQL 复制中源和副本服务器的基于位置的同步。

**表 14.28 位置同步函数**

| 名称 | 描述 | 引入版本 | 弃用版本 |
| --- | --- | --- | --- |
| `MASTER_POS_WAIT()` | 阻塞，直到副本已读取并应用了指定位置之前的所有更新 |  | 8.0.26 |
| `SOURCE_POS_WAIT()` | 阻塞，直到副本已读取并应用了指定位置之前的所有更新 | 8.0.26 |  |

+   [`MASTER_POS_WAIT(*`log_name`*,*`log_pos`*[,*`timeout`*][,*`channel`*])`](replication-functions-synchronization.html#function_master-pos-wait)

    这个函数用于控制源和副本同步。它会阻塞，直到副本已经读取并应用了源二进制日志中指定位置之前的所有更新。从 MySQL 8.0.26 开始，`MASTER_POS_WAIT()`已被弃用，应该使用别名`SOURCE_POS_WAIT()`。在 MySQL 8.0.26 之前的版本中，请使用`MASTER_POS_WAIT()`。

    返回值是副本需要等待的日志事件数量，以便前进到指定位置。如果复制 SQL 线程未启动、副本的源信息未初始化、参数不正确或发生错误，则函数返回`NULL`。如果超过超时时间，则返回`-1`。如果`MASTER_POS_WAIT()`等待时复制 SQL 线程停止，则函数返回`NULL`。如果副本已经超过指定位置，则函数立即返回。

    如果二进制日志文件位置被标记为无效，函数会等待直到知道有效的文件位置。当为复制通道设置了`CHANGE REPLICATION SOURCE TO`选项`GTID_ONLY`，并且服务器重新启动或复制停止时，二进制日志文件位置可以被标记为无效。在成功应用超过给定文件位置的事务后，文件位置变为有效。如果应用程序未达到指定位置，则函数会等待直到超时。使用`SHOW REPLICA STATUS`语句检查二进制日志文件位置是否被标记为无效。

    在多线程复制中，该函数会等待直到达到由`replica_checkpoint_group`、`slave_checkpoint_group`、`replica_checkpoint_period`或`slave_checkpoint_period`系统变量设置的限制时间，当调用检查点操作更新复制品状态时。根据系统变量的设置，该函数可能会在指定位置到达后的一段时间后返回。

    如果使用了二进制日志事务压缩，并且指定位置处的事务负载已被压缩（作为`Transaction_payload_event`），该函数会等待直到整个事务被读取和应用，并且位置已更新。

    如果指定了*`timeout`*值，`MASTER_POS_WAIT()`在经过*`timeout`*秒后停止等待。*`timeout`*必须大于或等于 0。（当服务器运行在严格 SQL 模式下时，负的*`timeout`*值会立即被拒绝，并显示`ER_WRONG_ARGUMENTS`；否则函数返回**`NULL`**，并发出警告。）

    可选的*`channel`*值使您能够命名函数应用于哪个复制通道。有关更多信息，请参见第 19.2.2 节，“复制通道”。

    此函数对于基于语句的复制是不安全的。如果在`binlog_format`设置为`STATEMENT`时使用此函数，将记录警告。

+   [`SOURCE_POS_WAIT(*`log_name`*,*`log_pos`*[,*`timeout`*][,*`channel`*])`](replication-functions-synchronization.html#function_source-pos-wait)

    此函数用于控制源-复制品同步。它会阻塞，直到复制品已读取并应用源二进制日志中指定位置的所有更新。从 MySQL 8.0.26 开始，使用`SOURCE_POS_WAIT()`代替从该版本开始弃用的`MASTER_POS_WAIT()`。在 MySQL 8.0.26 之前的版本中，请使用`MASTER_POS_WAIT()`。

    返回值是复制必须等待的日志事件数以前进到指定位置。如果复制 SQL 线程未启动，复制源信息未初始化，参数不正确或发生错误，则函数返回`NULL`。如果超时已超过，则返回`-1`。如果`SOURCE_POS_WAIT()`在等待时复制 SQL 线程停止，则函数返回`NULL`。如果复制已超过指定位置，则函数立即返回。

    如果二进制日志文件位置被标记为无效，该函数将等待直到已知有效文件位置。当为复制通道设置了`CHANGE REPLICATION SOURCE TO`选项`GTID_ONLY`，并且服务器重新启动或复制停止时，二进制日志文件位置可以被标记为无效。在成功应用超过给定文件位置的事务后，文件位置变为有效。如果应用程序未达到指定位置，则函数将等待超时。使用`SHOW REPLICA STATUS`语句检查二进制日志文件位置是否已标记为无效。

    在多线程复制中，当调用检查点操作以更新复制状态时，该函数将等待直到达到`replica_checkpoint_group`或`replica_checkpoint_period`系统变量设置的限制。根据系统变量的设置，因此该函数可能在达到指定位置后的一段时间后返回。

    如果使用二进制日志事务压缩，并且指定位置处的事务有效载荷已压缩（作为`Transaction_payload_event`），则函数将等待直到整个事务已被读取和应用，并且位置已更新。

    如果指定了*`timeout`*值，则`SOURCE_POS_WAIT()`在*`timeout`*秒已过时停止等待。*`timeout`*必须大于或等于 0。 （在严格 SQL 模式下，负的*`timeout`*值将立即被`ER_WRONG_ARGUMENTS`拒绝；否则函数返回`NULL`，并引发警告。）

    可选的*`channel`*值使您能够命名函数应用于哪个复制通道。有关更多信息，请参见 Section 19.2.2, “Replication Channels”。

    当`binlog_format`设置为`STATEMENT`时，此函数不适用于基于语句的复制，如果您在这种情况下使用此函数，将会记录警告。
