# 29.12.11 性能模式复制表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-tables.html)

29.12.11.1 复制连接配置表

29.12.11.2 复制连接状态表

29.12.11.3 复制异步连接故障转移表

29.12.11.4 复制异步连接故障转移管理表

29.12.11.5 复制应用程序配置表

29.12.11.6 复制应用程序状态表

29.12.11.7 按协调器复制应用程序状态表

29.12.11.8 按工作者复制应用程序状态表

29.12.11.9 全局复制应用程序过滤器表

29.12.11.10 复制应用程序过滤器表

29.12.11.11 复制组成员表

29.12.11.12 复制组成员统计表

29.12.11.13 复制组成员操作表

29.12.11.14 复制组配置版本表

29.12.11.15 复制组通信信息表

29.12.11.16 二进制日志事务压缩统计表

性能模式提供了暴露复制信息的表。这类似于`SHOW REPLICA STATUS`语句提供的信息，但以表格形式呈现更易访问，并具有可用性优势：

+   `SHOW REPLICA STATUS`输出对于视觉检查很有用，但对于程序化使用则不太方便。相比之下，使用性能模式表，关于复制状态的信息可以使用通用的`SELECT`查询进行搜索，包括复杂的`WHERE`条件、连接等。

+   查询结果可以保存在表中进行进一步分析，或分配给变量，从而在存储过程中使用。

+   复制表提供更好的诊断信息。对于多线程副本操作，`SHOW REPLICA STATUS`使用`Last_SQL_Errno`和`Last_SQL_Error`字段报告所有协调器和工作线程的错误，因此只有最近的错误可见，信息可能会丢失。复制表按线程基础存储错误，而不会丢失信息。

+   最后一次查看的事务在每个工作线程的复制表中可见。这是从`SHOW REPLICA STATUS`中无法获取的信息。

+   熟悉性能模式界面的开发人员可以通过向表中添加行来扩展复制表，以提供额外的信息。

#### 复制表描述

性能模式提供以下与复制相关的表：

+   包含有关副本与源之间连接信息的表：

    +   `replication_connection_configuration`: 连接到源的配置参数

    +   `replication_connection_status`: 与源的当前连接状态

    +   `replication_asynchronous_connection_failover`: 异步连接故障转移机制的源列表

+   包含有关事务应用程序的一般（非线程特定）信息的表：

    +   `replication_applier_configuration`: 副本上事务应用程序的配置参数。

    +   `replication_applier_status`: 副本上事务应用程序的当前状态。

+   包含有关负责应用来自源接收的事务的特定线程的信息的表：

    +   `replication_applier_status_by_coordinator`: 协调器线程的状态（除非副本是多线程的，否则为空）。

    +   `replication_applier_status_by_worker`: 应用程序线程或工作线程的状态（如果副本是多线程的）。

+   包含有关基于通道的复制过滤器的信息的表：

    +   `replication_applier_filters`: 提供有关在特定复制通道上配置的复制过滤器的信息。

    +   `replication_applier_global_filters`: 提供有关全局复制过滤器的信息，这些过滤器适用于所有复制通道。

+   包含有关组复制成员的信息的表：

    +   `replication_group_members`: 提供组成员的网络和状态信息。

    +   `replication_group_member_stats`: 提供有关组成员和他们参与的事务的统计信息。

    更多信息请参见 Section 20.4, “监控组复制”。

当性能模式被禁用时，以下性能模式复制表仍然会被填充：

+   `replication_connection_configuration`

+   `replication_connection_status`

+   `replication_asynchronous_connection_failover`

+   `replication_applier_configuration`

+   `replication_applier_status`

+   `replication_applier_status_by_coordinator`

+   `replication_applier_status_by_worker`

例外是复制表中的本地时间信息（事务的开始和结束时间戳）`replication_connection_status`，`replication_applier_status_by_coordinator`和`replication_applier_status_by_worker`。当性能模式被禁用时，不会收集此信息。

以下各节更详细地描述了每个复制表，包括`SHOW REPLICA STATUS`生成的列与复制表中出现相同信息的列之间的对应关系。

本介绍剩余部分将描述性能模式如何填充复制表以及`SHOW REPLICA STATUS`中的哪些字段在表中没有表示。

#### 复制表的生命周期

性能模式填充复制表如下：

+   在执行`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`之前，表格是空的。

+   在执行`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`之后，配置参数可以在表格中看到。此时，没有活动的复制线程，因此`THREAD_ID`列为`NULL`，`SERVICE_STATE`列的值为`OFF`。

+   在`START REPLICA`之后（或 MySQL 8.0.22 之前，`START SLAVE`），可以看到非`NULL`的`THREAD_ID`值。空闲或活动的线程具有`SERVICE_STATE`值为`ON`。连接到源的线程在建立连接时具有`CONNECTING`值，并且只要连接持续，之后为`ON`。

+   在`STOP REPLICA`之后，`THREAD_ID`列变为`NULL`，不再存在的线程的`SERVICE_STATE`列的值为`OFF`。

+   在`STOP REPLICA`或线程由于错误而停止后，表格将被保留。

+   `replication_applier_status_by_worker` 表仅在复制品以多线程模式运行时才非空。也就是说，如果 `replica_parallel_workers` 或 `slave_parallel_workers` 系统变量大于 0，则在执行 `START REPLICA` 时填充此表，并且行数显示工作线程数。

#### 复制品状态信息不在复制表中

性能模式复制表中的信息与 `SHOW REPLICA STATUS` 中可用的信息略有不同，因为这些表面向全局事务标识符（GTIDs）的使用，而不是文件名和位置，并且它们代表服务器 UUID 值，而不是服务器 ID 值。由于这些差异，性能模式复制表中未保留或以不同方式表示几个 `SHOW REPLICA STATUS` 列：

+   以下字段是指文件名和位置，不会被保留：

    ```sql
    Master_Log_File
    Read_Master_Log_Pos
    Relay_Log_File
    Relay_Log_Pos
    Relay_Master_Log_File
    Exec_Master_Log_Pos
    Until_Condition
    Until_Log_File
    Until_Log_Pos
    ```

+   `Master_Info_File` 字段不会被保留。它指的是用于复制品源元数据存储库的 `master.info` 文件，已被用于存储库的崩溃安全表所取代。

+   以下字段基于 `server_id`，而不是 `server_uuid`，并且不会被保留：

    ```sql
    Master_Server_Id
    Replicate_Ignore_Server_Ids
    ```

+   `Skip_Counter` 字段基于事件计数，而不是 GTIDs，并且不会被保留。

+   这些错误字段是 `Last_SQL_Errno` 和 `Last_SQL_Error` 的别名，因此不会被保留：

    ```sql
    Last_Errno
    Last_Error
    ```

    在性能模式中，此错误信息可在 `replication_applier_status_by_worker` 表的 `LAST_ERROR_NUMBER` 和 `LAST_ERROR_MESSAGE` 列中找到（如果复制品是多线程，则还可在 `replication_applier_status_by_coordinator` 中找到）。这些表提供比 `Last_Errno` 和 `Last_Error` 更具体的每个线程的错误信息。

+   提供有关命令行过滤选项的信息的字段不会被保留：

    ```sql
    Replicate_Do_DB
    Replicate_Ignore_DB
    Replicate_Do_Table
    Replicate_Ignore_Table
    Replicate_Wild_Do_Table
    Replicate_Wild_Ignore_Table
    ```

+   `Replica_IO_State` 和 `Replica_SQL_Running_State` 字段不会被保留。如果需要，可以通过使用适当复制表的 `THREAD_ID` 列并将其与 `INFORMATION_SCHEMA` `PROCESSLIST` 表中的 `ID` 列连接，以选择后者表的 `STATE` 列来从进程列表中获取这些值。

+   `Executed_Gtid_Set` 字段可能显示一个包含大量文本的大集合。相反，性能模式表显示当前正在被副本应用的事务的 GTID。或者，已执行的 GTID 集合可以从 `gtid_executed` 系统变量的值中获取。

+   `Seconds_Behind_Master` 和 `Relay_Log_Space` 字段处于待定状态，不会被保留。

#### 复制通道

复制性能模式表的第一列是 `CHANNEL_NAME`。这使得可以按照复制通道查看表。在非多源复制设置中，存在一个单一的默认复制通道。当在副本上使用多个复制通道时，可以按照复制通道过滤表以监视特定的复制通道。有关更多信息，请参见 Section 19.2.2, “Replication Channels” 和 Section 19.1.5.8, “Monitoring Multi-Source Replication”。
