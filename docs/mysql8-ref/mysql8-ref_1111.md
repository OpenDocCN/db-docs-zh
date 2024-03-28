> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-replica-status.html`](https://dev.mysql.com/doc/refman/8.0/en/show-replica-status.html)

#### 15.7.7.35 SHOW REPLICA STATUS Statement

```sql
SHOW {REPLICA | SLAVE} STATUS [FOR CHANNEL *channel*]
```

此语句提供有关复制线程的关键参数的状态信息。从 MySQL 8.0.22 开始，使用`SHOW REPLICA STATUS`代替`SHOW SLAVE STATUS`，该语句从该版本开始已弃用。在 MySQL 8.0.22 之前的版本中，请使用`SHOW SLAVE STATUS`。该语句需要`REPLICATION CLIENT`权限（或已弃用的`SUPER`权限）。

`SHOW REPLICA STATUS`是非阻塞的。与`STOP REPLICA`同时运行时，`SHOW REPLICA STATUS`会立即返回，而不会等待`STOP REPLICA`完成关闭复制 SQL（应用程序）线程或复制 I/O（接收器）线程（或两者）。这允许在监控和其他应用程序中使用`SHOW REPLICA STATUS`，其中从`SHOW REPLICA STATUS`获得即时响应比确保返回最新数据更重要。在 MySQL 8.0.22 中，SLAVE 关键字被 REPLICA 替换。

如果您使用**mysql**客户端发出此语句，可以使用`\G`语句终止符，而不是分号，以获得更易读的垂直布局：

```sql
mysql> SHOW REPLICA STATUS\G
*************************** 1\. row ***************************
             Replica_IO_State: Waiting for source to send event
                  Source_Host: 127.0.0.1
                  Source_User: root
                  Source_Port: 13000
                Connect_Retry: 1
              Source_Log_File: master-bin.000001
          Read_Source_Log_Pos: 927
               Relay_Log_File: slave-relay-bin.000002
                Relay_Log_Pos: 1145
        Relay_Source_Log_File: master-bin.000001
           Replica_IO_Running: Yes
          Replica_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Source_Log_Pos: 927
              Relay_Log_Space: 1355
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Source_SSL_Allowed: No
           Source_SSL_CA_File:
           Source_SSL_CA_Path:
              Source_SSL_Cert:
            Source_SSL_Cipher:
               Source_SSL_Key:
        Seconds_Behind_Source: 0
Source_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Source_Server_Id: 1
                  Source_UUID: 73f86016-978b-11ee-ade5-8d2a2a562feb
             Source_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
    Replica_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Source_Retry_Count: 10
                  Source_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Source_SSL_Crl:
           Source_SSL_Crlpath:
           Retrieved_Gtid_Set: 73f86016-978b-11ee-ade5-8d2a2a562feb:1-3
            Executed_Gtid_Set: 73f86016-978b-11ee-ade5-8d2a2a562feb:1-3
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Source_TLS_Version:
       Source_public_key_path:
        Get_Source_public_key: 0
            Network_Namespace:
```

性能模式提供了暴露复制信息的表。这类似于从`SHOW REPLICA STATUS`语句中获取的信息，但以表格形式表示。有关详细信息，请参阅第 29.12.11 节，“性能模式复制表”。

从 MySQL 8.0.27 开始，您可以在`CHANGE REPLICATION SOURCE TO`语句上设置`GTID_ONLY`选项，以阻止复制通道在复制元数据存储库中持久化文件名和文件位置。使用此设置，源二进制日志文件和中继日志文件的文件位置将在内存中跟踪。`SHOW REPLICA STATUS`语句在正常使用中仍会显示文件位置。然而，由于文件位置在连接元数据存储库和应用程序元数据存储库中除了在少数情况下不会定期更新，如果服务器重新启动，它们可能会过时。

对于在服务器启动后具有`GTID_ONLY`设置的复制通道，源二进制日志文件的读取和应用文件位置（`Read_Source_Log_Pos`和`Exec_Source_Log_Pos`）设置为零，并且文件名（`Source_Log_File`和`Relay_Source_Log_File`）设置为`INVALID`。中继日志文件名（`Relay_Log_File`）根据 relay_log_recovery 设置进行设置，可以是在服务器启动时创建的新文件，也可以是第一个中继日志文件。文件位置（`Relay_Log_Pos`）设置为位置 4，并且使用 GTID 自动跳过来跳过文件中已经应用的任何事务。

当接收器线程联系源并获取有效位置信息时，读取位置（`Read_Source_Log_Pos`）和文件名（`Source_Log_File`）将更新为正确的数据并变为有效。当应用程序线程应用来自源的事务，或跳过已执行的事务时，执行位置（`Exec_Source_Log_Pos`）和文件名（`Relay_Source_Log_File`）将更新为正确的数据并变为有效。中继日志文件位置（`Relay_Log_Pos`）也在那时更新。

以下列表描述了`SHOW REPLICA STATUS`返回的字段。有关解释其含义的更多信息，请参见第 19.1.7.1 节，“检查复制状态”。

+   `Replica_IO_State`

    复制`SHOW PROCESSLIST`输出的`State`字段，用于复制 I/O（接收器）线程。这告诉您线程正在做什么：尝试连接到源，等待来自源的事件，重新连接到源等等。有关可能状态的列表，请参见第 10.14.5 节，“复制 I/O（接收器）线程状态” Thread States")。

+   `Source_Host`

    复制品连接到的源主机。

+   `Source_User`

    用于连接到源的帐户的用户名。

+   `Source_Port`

    用于连接到源的端口。

+   `Connect_Retry`

    连接重试之间的秒数（默认为 60）。可以使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）进行设置。

+   `Source_Log_File`

    当 I/O（接收器）线程当前正在读取的源二进制日志文件的名称。对于在服务器启动后具有`GTID_ONLY`设置的复制通道，此设置为`INVALID`。当复制品联系源时，它将被更新。

+   `Read_Source_Log_Pos`

    I/O（接收器）线程已读取的当前源二进制日志文件中的位置。对于具有`GTID_ONLY`设置的复制通道，在服务器启动后，此设置将设置为零。当副本联系源时，它将被更新。

+   `Relay_Log_File`

    SQL（应用程序）线程当前正在读取和执行的中继日志文件的名称。

+   `Relay_Log_Pos`

    SQL（应用程序）线程已读取和执行的当前中继日志文件中的位置。

+   `Relay_Source_Log_File`

    源二进制日志文件的名称，其中包含 SQL（应用程序）线程执行的最新事件。对于具有`GTID_ONLY`设置的复制通道，在服务器启动后，此设置将设置为`INVALID`。当执行或跳过事务时，它将被更新。

+   `Replica_IO_Running`

    复制 I/O（接收器）线程是否已启动并已成功连接到源。在内部，此线程的状态由以下三个值之一表示：

    +   **MYSQL_REPLICA_NOT_RUN. ** 复制 I/O（接收器）线程未运行。对于此状态，`Replica_IO_Running`为`No`。

    +   **MYSQL_REPLICA_RUN_NOT_CONNECT. ** 复制 I/O（接收器）线程正在运行，但未连接到复制源。对于此状态，`Replica_IO_Running`为`Connecting`。

    +   **MYSQL_REPLICA_RUN_CONNECT. ** 复制 I/O（接收器）线程正在运行，并且已连接到复制源。对于此状态，`Replica_IO_Running`为`Yes`。

+   `Replica_SQL_Running`

    复制 SQL（应用程序）线程是否已启动。

+   `Replicate_Do_DB`, `Replicate_Ignore_DB`

    使用`--replicate-do-db`和`--replicate-ignore-db`选项或`CHANGE REPLICATION FILTER`语句指定的任何数据库的名称。如果使用了`FOR CHANNEL`子句，则显示特定通道的复制过滤器。否则，显示每个复制通道的复制过滤器。

+   `Replicate_Do_Table`, `Replicate_Ignore_Table`, `Replicate_Wild_Do_Table`, `Replicate_Wild_Ignore_Table`

    任何使用 `--replicate-do-table`、`--replicate-ignore-table`、`--replicate-wild-do-table` 和 `--replicate-wild-ignore-table` 选项或 `CHANGE REPLICATION FILTER` 语句指定的表的名称。如果使用了 `FOR CHANNEL` 子句，则显示特定通道的复制过滤器。否则，显示每个复制通道的复制过滤器。

+   `Last_Errno`，`Last_Error`

    这些列是 `Last_SQL_Errno` 和 `Last_SQL_Error` 的别名。

    执行 `RESET MASTER` 或 `RESET REPLICA` 会重置这些列中显示的值。

    注意

    当复制 SQL 线程收到错误时，首先报告错误，然后停止 SQL 线程。这意味着在 `SHOW REPLICA STATUS` 显示 `Last_SQL_Errno` 的值为非零时，`Replica_SQL_Running` 仍显示 `Yes`，存在一个很小的时间窗口。

+   `Skip_Counter`

    `sql_slave_skip_counter` 系统变量的当前值。参见 SET GLOBAL sql_slave_skip_counter Syntax。

+   `Exec_Source_Log_Pos`

    复制 SQL 线程已读取和执行的当前源二进制日志文件中的位置，标记下一个要处理的事务或事件的开始。对于具有 `GTID_ONLY` 设置的复制通道，此值在服务器启动后设置为零。当执行或跳过事务时，它将被更新。

    当从现有副本开始新建副本时，可以使用此值与 `CHANGE REPLICATION SOURCE TO` 语句的 `SOURCE_LOG_POS` 选项（从 MySQL 8.0.23 开始）或 `CHANGE MASTER TO` 语句的 `MASTER_LOG_POS` 选项（MySQL 8.0.23 之前）一起使用，以便新副本从此处读取。源二进制日志中的 (`Relay_Source_Log_File`, `Exec_Source_Log_Pos`) 给出的坐标对应于中继日志中的 (`Relay_Log_File`, `Relay_Log_Pos`) 给出的坐标。

    从已执行的中继日志中的事务序列中的不一致性可能导致此值成为“低水位标记”。换句话说，在该位置之前出现的事务已经提交，但在该位置之后的事务可能已经提交或未提交。如果需要纠正这些间隙，请使用`START REPLICA UNTIL SQL_AFTER_MTS_GAPS`。有关更多信息，请参��Section 19.5.1.34, “Replication and Transaction Inconsistencies”。

+   `Relay_Log_Space`

    所有现有中继日志文件的总合大小。

+   `Until_Condition`、`Until_Log_File`、`Until_Log_Pos`

    `START REPLICA`语句中`UNTIL`子句中指定的值。

    `Until_Condition` 有以下值：

    +   如果未指定`UNTIL`子句，则为`None`。

    +   `Source` 如果复制品正在读取直到源的二进制日志中的特定位置。

    +   `Relay` 如果复制品正在读取直到其中继日志中的特定位置。

    +   `SQL_BEFORE_GTIDS` 如果复制 SQL 线程正在处理事务，直到达到`gtid_set`中列出的第一个事务。

    +   `SQL_AFTER_GTIDS` 如果复制线程正在处理直到`gtid_set`中的最后一个事务被两个线程都处理完。

    +   `SQL_AFTER_MTS_GAPS` 如果多线程复制品的 SQL 线程正在运行，直到在中继日志中不再找到间隙为止。

    `Until_Log_File` 和 `Until_Log_Pos` 指示定义复制 SQL 线程停止执行的坐标的日志文件名和位置。

    有关`UNTIL`子句的更多信息，请参见 Section 15.4.2.7, “START SLAVE Statement”。

+   `Source_SSL_Allowed`、`Source_SSL_CA_File`、`Source_SSL_CA_Path`、`Source_SSL_Cert`、`Source_SSL_Cipher`、`Source_SSL_CRL_File`、`Source_SSL_CRL_Path`、`Source_SSL_Key`、`Source_SSL_Verify_Server_Cert`

    这些字段显示了复制品用于连接到源的 SSL 参数（如果有）。

    `Source_SSL_Allowed` 有以下值：

    +   如果允许与源建立 SSL 连接，则为`Yes`。

    +   如果不允许与源建立 SSL 连接，则为`No`。

    +   如果允许 SSL 连接但复制品服务器未启用 SSL 支持，则为`Ignored`。

    其他与 SSL 相关字段的值对应于`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）的`SOURCE_SSL_*`选项的值，或者`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）的`MASTER_SSL_*`选项的值。请参见 Section 15.4.2.1, “CHANGE MASTER TO Statement”。

+   `Seconds_Behind_Source`

    该字段表示副本的“延迟”程度：

    +   当副本正在处理更新时，此字段显示副本上当前时间戳与源上记录的当前正在处理的事件的原始时间戳之间的差异。

    +   当副本当前没有处理任何事件时，此值为 0。

    本质上，该字段衡量了复制 SQL（应用程序）线程和复制 I/O（接收器）线程之间的时间差（以秒为单位）。如果源和副本之间的网络连接速度很快，复制接收线程与源之间非常接近，因此该字段很好地近似了复制应用程序线程相对于源的延迟。如果网络速度慢，这*不*是一个很好的近似值；复制应用程序线程可能经常赶上读取速度慢的复制接收线程，因此`Seconds_Behind_Source`经常显示为 0，即使复制接收线程相对于源来说是延迟的。换句话说，*此列仅适用于快速网络*。

    即使源和副本的时钟时间不相同，只要在副本接收线程启动时计算的差异保持不变，这种时间差计算也能正常工作。任何更改，包括 NTP 更新，都可能导致时钟偏差，从而使`Seconds_Behind_Source`的计算不太可靠。

    在 MySQL 8.0 中，如果复制应用程序线程未运行，或者应用程序线程已消耗完中继日志且复制接收线程未运行，则此字段为`NULL`（未定义或未知）。（在旧版本的 MySQL 中，如果复制应用程序线程或复制接收线程未运行或未连接到源，则此字段为`NULL`。）如果复制接收线程正在运行但中继日志已用尽，则`Seconds_Behind_Source`设置为 0。

    `Seconds_Behind_Source`的值基于事件中存储的时间戳，这些时间戳通过复制进行保留。这意味着如果源 M1 本身是 M0 的副本，那么来自 M1 二进制日志的任何事件，其来源于 M0 的二进制日志，都具有该事件的 M0 时间戳。这使得 MySQL 能够成功复制`TIMESTAMP`。然而，对于`Seconds_Behind_Source`的问题在于，如果 M1 还接收来自客户端的直接更新，那么`Seconds_Behind_Source`的值会随机波动，因为有时来自 M1 的最后一个事件源自 M0，有时是 M1 上的直接更新的结果。

    当使用多线程副本时，应注意此值基于`Exec_Source_Log_Pos`，因此可能不反映最近提交事务的位置。

+   `Last_IO_Errno`，`Last_IO_Error`

    导致复制 I/O（接收器）线程停止的最近错误的错误编号和错误消息。错误编号为 0，消息为空字符串表示“无错误”。如果`Last_IO_Error`值不为空，则错误值也会出现在副本的错误日志中。

    I/O 错误信息包括一个时间戳，显示最近一次 I/O（接收器）线程错误发生的时间。这个时间戳使用格式*`YYMMDD hh:mm:ss`*，并显示在`Last_IO_Error_Timestamp`列中。

    发出`RESET MASTER`或`RESET REPLICA`将重置这些列中显示的值。

+   `Last_SQL_Errno`，`Last_SQL_Error`

    导致复制 SQL（应用程序）线程停止的最近错误的错误编号和错误消息。错误编号为 0，消息为空字符串表示“无错误”。如果`Last_SQL_Error`值不为空，则错误值也会出现在副本的错误日志中。

    如果副本是多线程的，则复制 SQL 线程是工作线程的协调员。在这种情况下，`Last_SQL_Error`字段显示的内容与性能模式`replication_applier_status_by_coordinator`表中的`Last_Error_Message`列显示的内容完全相同。该字段值被修改以暗示其他工作线程可能存在更多故障，这可以在显示每个工作线程状态的`replication_applier_status_by_worker`表中看到。如果该表不可用，则可以使用副本错误日志。日志或`replication_applier_status_by_worker`表还应用于了解由`SHOW REPLICA STATUS`或协调员表显示的故障的更多信息。

    SQL 错误信息包括一个时间戳，显示最近一次 SQL（应用程序）线程错误发生的时间。这个时间戳使用格式*`YYMMDD hh:mm:ss`*，并显示在`Last_SQL_Error_Timestamp`列中。

    发出`RESET MASTER`或`RESET REPLICA`将重置这些列中显示的值。

    在 MySQL 8.0 中，`Last_SQL_Errno`和`Last_SQL_Error`列中显示的所有错误代码和消息对应于服务器错误消息参考中列出的错误值。在以前的版本中，这并不总是正确的。（Bug #11760365，Bug #52768）

+   `Replicate_Ignore_Server_Ids`

    任何已经使用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句的`IGNORE_SERVER_IDS`选项指定的服务器 ID，以便复制品忽略来自这些服务器的事件。在循环或其他多源复制设置中，当其中一个服务器被移除时，会使用此选项。如果以这种方式设置了任何服务器 ID，则会显示一个逗号分隔的一个或多个数字的列表。如果没有设置任何服务器 ID，则该字段为空。

    注意

    `slave_master_info`表中的`Ignored_server_ids`值还显示要忽略的服务器 ID，但作为一个以空格分隔的列表，前面是要忽略的服务器 ID 总数。例如，如果发出包含`IGNORE_SERVER_IDS = (2,6,9)`选项的`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句，告诉复制品忽略具有服务器 ID 2、6 或 9 的源，那么该信息显示如下：

    ```sql
     Replicate_Ignore_Server_Ids: 2, 6, 9
    ```

    ```sql
     Ignored_server_ids: 3, 2, 6, 9
    ```

    `Replicate_Ignore_Server_Ids`过滤是由 I/O（接收器）线程执行的，而不是由 SQL（应用程序）线程执行的，这意味着被过滤掉的事件不会被写入中继日志。这与服务器选项`--replicate-do-table`采取的过滤操作不同，后者适用于应用程序线程。

    注意

    从 MySQL 8.0 开始，如果在任何通道具有使用`IGNORE_SERVER_IDS`设置的现有服务器 ID 时发出`SET GTID_MODE=ON`，则会发出弃用警告。在启动基于 GTID 的复制之前，使用`SHOW REPLICA STATUS`检查并清除涉及服务器上的所有被忽略的服务器 ID 列表。您可以通过发出包含空列表的`IGNORE_SERVER_IDS`选项的`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句来清除列表。

+   `Source_Server_Id`

    来自源的`server_id`值。

+   `Source_UUID`

    来自源的`server_uuid`值。

+   `Source_Info_File`

    `master.info`文件的位置，现在已经不推荐使用。从 MySQL 8.0 开始，默认情况下，表用于复制品的连接元数据存储库。

+   `SQL_Delay`

    复制品必须滞后源的秒数。

+   `SQL_Remaining_Delay`

    当`Replica_SQL_Running_State`为`Waiting until MASTER_DELAY seconds after source executed event`时，此字段包含剩余的延迟秒数。在其他时间，此字段为`NULL`。

+   `Replica_SQL_Running_State`

    SQL 线程的状态（类似于`Replica_IO_State`）。该值与通过`SHOW PROCESSLIST`显示的 SQL 线程的`State`值相同。第 10.14.6 节，“复制 SQL 线程状态”提供了可能状态的列表。

+   `Source_Retry_Count`

    复制品在连接丢失的情况下可以尝试重新连接到源的次数。可以使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）的`SOURCE_RETRY_COUNT` | `MASTER_RETRY_COUNT`选项或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）的选项来设置此值，或者使用旧的`--master-retry-count`服务器选项（仍然支持向后兼容性）。

+   `Source_Bind`

    如果有的话，复制品绑定到的网络接口。这是使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）的`SOURCE_BIND` | `MASTER_BIND`选项设置的。

+   `Last_IO_Error_Timestamp`

    以*`YYMMDD hh:mm:ss`*格式表示的时间戳，显示最近一次 I/O 错误发生的时间。

+   `Last_SQL_Error_Timestamp`

    以*`YYMMDD hh:mm:ss`*格式表示的时间戳，显示最近一次 SQL 错误发生的时间。

+   `Retrieved_Gtid_Set`

    对应于此复制品接收的所有事务的全局事务 ID 集合。如果不使用 GTID，则为空。有关更多信息，请参见 GTID Sets。

    这是存在或曾经存在于中继日志中的所有 GTID 的集合。每个 GTID 在接收到`Gtid_log_event`时立即添加。这可能导致部分传输的事务的 GTID 被包含在集合中。

    当所有中继日志因执行`RESET REPLICA`或`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`，或由`--relay-log-recovery`选项的影响而丢失时，集合将被清除。当`relay_log_purge = 1`时，始终保留最新的中继日志，并且集合不会被清除。

+   `Executed_Gtid_Set`

    写入二进制日志的全局事务 ID 集。这与此服务器上全局 `gtid_executed` 系统变量的值相同，以及此服务器上 `SHOW MASTER STATUS` 输出中的 `Executed_Gtid_Set` 的值。如果未使用 GTID，则为空。查看 GTID 集获取更多信息。

+   `Auto_Position`

    如果通道使用 GTID 自动定位，则为 1，否则为 0。

+   `Replicate_Rewrite_DB`

    `Replicate_Rewrite_DB` 值显示指定的任何复制过滤规则。例如，如果设置了以下复制过滤规则：

    ```sql
    CHANGE REPLICATION FILTER REPLICATE_REWRITE_DB=((db1,db2), (db3,db4));
    ```

    `Replicate_Rewrite_DB` 值显示：

    ```sql
    Replicate_Rewrite_DB: (db1,db2),(db3,db4)
    ```

    有关更多信息，请参阅第 15.4.2.2 节，“CHANGE REPLICATION FILTER Statement”。

+   `Channel_name`

    正在显示的复制通道。始终存在一个默认的复制通道，可以添加更多复制通道。查看第 19.2.2 节，“复制通道”获取更多信息。

+   `Master_TLS_Version`

    源使用的 TLS 版本。有关 TLS 版本信息，请参阅第 8.3.2 节，“加密连接 TLS 协议和密码”。

+   `Source_public_key_path`

    文件路径名，其中包含源所需的用于 RSA 密钥对密码交换的副本端的公钥文件。文件必须采用 PEM 格式。此列适用于使用 `sha256_password` 或 `caching_sha2_password` 认证插件进行身份验证的副本。

    如果给定 `Source_public_key_path` 并指定有效的公钥文件，则优先于 `Get_source_public_key`。

+   `Get_source_public_key`

    是否从源请求基于 RSA 密钥对的密码交换所需的公钥。此列适用于使用 `caching_sha2_password` 认证插件进行身份验证的副本。对于该插件，除非请求，否则源不会发送公钥。

    如果给定 `Source_public_key_path` 并指定有效的公钥文件，则优先于 `Get_source_public_key`。

+   `Network_Namespace`

    网络命名空间名称；如果连接使用默认（全局）命名空间，则为空。有关网络命名空间的信息，请参阅第 7.1.14 节，“网络命名空间支持”。此列在 MySQL 8.0.22 中添加。
