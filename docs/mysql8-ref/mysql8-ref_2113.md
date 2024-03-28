> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-processlist-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-processlist-table.html)

#### 29.12.21.7 进程列表表

MySQL 进程列表指示服务器内执行的一组线程当前正在执行的操作。`processlist` 表是进程信息的一个来源。有关此表与其他来源的比较，请参见进程信息的来源。

`processlist` 表可以直接查询。如果您拥有 `PROCESS` 权限，您可以看到所有线程，甚至属于其他用户的线程。否则（没有 `PROCESS` 权限），非匿名用户可以访问有关自己线程的信息，但不能访问其他用户的线程，匿名用户无法访问线程信息。

注意

如果启用了 `performance_schema_show_processlist` 系统变量，则 `processlist` 表还作为 `SHOW PROCESSLIST` 语句底层的替代实现的基础。有关详细信息，请参见本节后面的内容。

`processlist` 表包含每个服务器进程的一行：

```sql
mysql> SELECT * FROM performance_schema.processlist\G
*************************** 1\. row ***************************
     ID: 5
   USER: event_scheduler
   HOST: localhost
     DB: NULL
COMMAND: Daemon
   TIME: 137
  STATE: Waiting on empty queue
   INFO: NULL
*************************** 2\. row ***************************
     ID: 9
   USER: me
   HOST: localhost:58812
     DB: NULL
COMMAND: Sleep
   TIME: 95
  STATE:
   INFO: NULL
*************************** 3\. row ***************************
     ID: 10
   USER: me
   HOST: localhost:58834
     DB: test
COMMAND: Query
   TIME: 0
  STATE: executing
   INFO: SELECT * FROM performance_schema.processlist
...
```

`processlist` 表具有以下列：

+   `ID`

    连接标识符。这是在 `SHOW PROCESSLIST` 语句的 `Id` 列中显示的相同值，在性能模式 `threads` 表的 `PROCESSLIST_ID` 列中显示，并在线程内由 `CONNECTION_ID()` 函数返回。

+   `USER`

    发出语句的 MySQL 用户。`system user`的值指的是服务器生成的用于内部处理任务的非客户端线程，例如延迟行处理程序线程或在副本主机上使用的 I/O 或 SQL 线程。对于`system user`，在`Host`列中没有指定主机。`unauthenticated user`指的是已经与客户端连接关联但客户端用户尚未进行身份验证的线程。`event_scheduler`指的是监视计划事件的线程（参见 Section 27.4, “Using the Event Scheduler”）。

    注意

    `system user`的`USER`值与`SYSTEM_USER`权限是不同的。前者指的是内部线程。后者区分系统用户和常规用户帐户类别（参见 Section 8.2.11, “Account Categories”）。

+   `HOST`

    发出语句的客户端主机名（除了`system user`外，没有主机名）。对于 TCP/IP 连接，主机名以`*`host_name`*:*`client_port`*`格式报告，以便更容易确定哪个客户端在执行什么操作。

+   `DB`

    线程的默认数据库，如果没有选择任何数据库，则为`NULL`。

+   `COMMAND`

    线程代表客户端执行的命令类型，或者如果会话处于空闲状态，则为`Sleep`。有关线程命令的描述，请参见 Section 10.14, “Examining Server Thread (Process) Information” Information")。此列的值对应于客户端/服务器协议的`COM_*`xxx`*`命令和`Com_*`xxx`*`状态变量。参见 Section 7.1.10, “Server Status Variables”。

+   `TIME`

    线程在当前状态下已经持续的时间（以秒为单位）。对于副本 SQL 线程，该值是最后一个复制事件的时间戳与副本主机实际时间之间的秒数。参见 Section 19.2.3, “Replication Threads”。

+   `STATE`

    指示线程正在执行的操作、事件或状态。有关`STATE`值的描述，请参见 Section 10.14, “Examining Server Thread (Process) Information” Information")。

    大多数状态对应非常快速的操作。如果一个线程在特定状态下停留了很多秒，可能存在需要调查的问题。

+   `INFO`

    线程正在执行的语句，如果未执行任何语句则为`NULL`。该语句可能是发送到服务器的语句，或者如果该语句执行其他语句，则为最内层语句。例如，如果`CALL`语句执行一个正在执行`SELECT`语句的存储过程，则`INFO`值显示`SELECT`语句。

+   `EXECUTION_ENGINE`

    查询执行引擎。该值可以是`PRIMARY`或`SECONDARY`。用于 MySQL HeatWave 服务和 HeatWave，其中`PRIMARY`引擎是`InnoDB`，`SECONDARY`引擎是 HeatWave（`RAPID`）。对于 MySQL 社区版服务器、MySQL 企业版服务器（本地）以及没有 HeatWave 的 MySQL HeatWave 服务，该值始终为`PRIMARY`。此列在 MySQL 8.0.29 中添加。

`processlist`表具有以下索引：

+   主键为(`ID`)

`TRUNCATE TABLE`不允许用于`processlist`表。

如前所述，如果启用了`performance_schema_show_processlist`系统变量，则`processlist`表作为其他进程信息源的替代实现的基础：

+   `SHOW PROCESSLIST`语句。

+   **mysqladmin processlist**命令（使用`SHOW PROCESSLIST`语句）。

默认的`SHOW PROCESSLIST`实现在持有全局互斥锁的同时从线程管理器中遍历活动线程。这会对性能产生负面影响，特别是在繁忙系统上。另一种`SHOW PROCESSLIST`实现基于性能模式`processlist`表。该实现从性能模式而不是线程管理器查询活动线程数据，并且不需要互斥锁。

MySQL 配置会影响`processlist`表的内容如下：

+   最低要求配置：

    +   MySQL 服务器必须配置并构建为启用线程仪表。这是默认情况；可以使用`DISABLE_PSI_THREAD` **CMake**选项进行控制。

    +   性能模式必须在服务器启动时启用。这是默认情况；可以使用`performance_schema`系统变量来控制。

    在满足该配置的情况下，`performance_schema_show_processlist`启用或禁用替代的`SHOW PROCESSLIST`实现。如果未满足最低配置要求，`processlist`表（因此`SHOW PROCESSLIST`）可能无法返回所有数据。

+   推荐配置：

    +   为了避免一些线程被忽略：

        +   将`performance_schema_max_thread_instances`系统变量设置为默认值或至少设置为与`max_connections`系统变量一样大。

        +   将`performance_schema_max_thread_classes`系统变量设置为默认值。

    +   为了避免一些`STATE`列值为空，请将`performance_schema_max_stage_classes`系统变量设置为默认值。

    这些配置参数的默认值为`-1`，这会导致性能模式在服务器启动时自动调整它们的大小。使用指定的参数设置，`processlist`表（因此`SHOW PROCESSLIST`）会生成完整的进程信息。

前述配置参数会影响`processlist`表的内容。然而，对于给定的配置，`processlist`的内容不受`performance_schema_show_processlist`设置的影响。

替代的进程列表实现不适用于`INFORMATION_SCHEMA`的`PROCESSLIST`表或 MySQL 客户端/服务器协议的`COM_PROCESS_INFO`命令。
