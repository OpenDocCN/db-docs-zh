> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-threads-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-threads-table.html)

#### 29.12.21.8 线程表

`threads`表包含每个服务器线程的一行。每行包含有关线程的信息，并指示是否为其启用了监控和历史事件记录：

```sql
mysql> SELECT * FROM performance_schema.threads\G
*************************** 1\. row ***************************
            THREAD_ID: 1
                 NAME: thread/sql/main
                 TYPE: BACKGROUND
       PROCESSLIST_ID: NULL
     PROCESSLIST_USER: NULL
     PROCESSLIST_HOST: NULL
       PROCESSLIST_DB: mysql
  PROCESSLIST_COMMAND: NULL
     PROCESSLIST_TIME: 418094
    PROCESSLIST_STATE: NULL
     PROCESSLIST_INFO: NULL
     PARENT_THREAD_ID: NULL
                 ROLE: NULL
         INSTRUMENTED: YES
              HISTORY: YES
      CONNECTION_TYPE: NULL
         THREAD_OS_ID: 5856
       RESOURCE_GROUP: SYS_default
     EXECUTION_ENGINE: PRIMARY
    CONTROLLED_MEMORY: 1456
MAX_CONTROLLED_MEMORY: 67480
         TOTAL_MEMORY: 1270430
     MAX_TOTAL_MEMORY: 1307317
     TELEMETRY_ACTIVE: NO
...
```

当性能模式初始化时，它基于当时存在的线程填充`threads`表。此后，每次服务器创建线程时都会添加新行。

新线程的`INSTRUMENTED`和`HISTORY`列值由`setup_actors`表的内容确定。有关如何使用`setup_actors`表来控制这些列的信息，请参阅 Section 29.4.6, “Pre-Filtering by Thread”.

从`threads`表中删除行是在线程结束时发生的。对于与客户端会话关联的线程，当会话结束时会发生删除。如果客户端启用了自动重新连接，并且会话在断开连接后重新连接，则会话将与具有不同`PROCESSLIST_ID`值的`threads`表中的新行相关联。新线程的初始`INSTRUMENTED`和`HISTORY`值可能与原始线程的值不同：`setup_actors`表可能已经发生了变化，如果在初始化行之后更改了原始线程的`INSTRUMENTED`或`HISTORY`值，则该更改不会传递到新线程。

您可以启用或禁用线程监控（即线程执行的事件是否被检测）和历史事件记录。要控制新前台线程的初始`INSTRUMENTED`和`HISTORY`值，请使用`setup_actors`表。要控制现有线程的这些方面，请设置`threads`表行的`INSTRUMENTED`和`HISTORY`列。（有关线程监控和历史事件记录发生条件的更多信息，请参阅`INSTRUMENTED`和`HISTORY`列的描述。）

与具有`PROCESSLIST_`前缀名称的`threads`表列进行比较，以及其他进程信息来源，请参阅进程信息来源。

重要

除了`threads`表之外的线程信息来源，只有当前用户具有`PROCESS`权限时，才会显示其他用户的线程信息。对于`threads`表并非如此；对于具有表的`SELECT`权限的任何用户，都会显示所有行。不应授予对`threads`表的`SELECT`权限的用户，不应该能够通过访问该表来查看其他用户的线程。

`threads`表具有以下列：

+   `THREAD_ID`

    唯一线程标识符。

+   `名称`

    与服务器中的线程仪表代码相关联的名称。例如，`thread/sql/one_connection`对应于负责处理用户连接的代码中的线程函数，而`thread/sql/main`代表服务器的`main()`函数。

+   `类型`

    线程类型，可以是`FOREGROUND`或`BACKGROUND`。用户连接线程是前台线程。与内部服务器活动相关的线程是后台线程。例如，内部`InnoDB`线程，“binlog dump”线程向副本发送信息，以及复制 I/O 和 SQL 线程。

+   `PROCESSLIST_ID`

    对于前台线程（与用户连接相关），这是连接标识符。这与`INFORMATION_SCHEMA` `PROCESSLIST`表的`ID`列中显示的值相同，在`SHOW PROCESSLIST`输出中显示，在线程内部由`CONNECTION_ID()`函数返回。

    对于后台线程（与用户连接无关），`PROCESSLIST_ID`为`NULL`，因此值不是唯一的。

+   `PROCESSLIST_USER`

    与前台线程关联的用户，对于后台线程为`NULL`。

+   `PROCESSLIST_HOST`

    与前台线程关联的客户端的主机名，对于后台线程为`NULL`。

    与`INFORMATION_SCHEMA` `PROCESSLIST`表的`HOST`列或`SHOW PROCESSLIST`输出的`Host`列不同，`PROCESSLIST_HOST`列不包括 TCP/IP 连接的端口号。要从性能模式获取此信息，请启用套接字检测（默认情况下未启用），并检查`socket_instances`表：

    ```sql
    mysql> SELECT NAME, ENABLED, TIMED
           FROM performance_schema.setup_instruments
           WHERE NAME LIKE 'wait/io/socket%';
    +----------------------------------------+---------+-------+
    | NAME                                   | ENABLED | TIMED |
    +----------------------------------------+---------+-------+
    | wait/io/socket/sql/server_tcpip_socket | NO      | NO    |
    | wait/io/socket/sql/server_unix_socket  | NO      | NO    |
    | wait/io/socket/sql/client_connection   | NO      | NO    |
    +----------------------------------------+---------+-------+
    3 rows in set (0.01 sec)

    mysql> UPDATE performance_schema.setup_instruments
           SET ENABLED='YES'
           WHERE NAME LIKE 'wait/io/socket%';
    Query OK, 3 rows affected (0.00 sec)
    Rows matched: 3  Changed: 3  Warnings: 0

    mysql> SELECT * FROM performance_schema.socket_instances\G
    *************************** 1\. row ***************************
               EVENT_NAME: wait/io/socket/sql/client_connection
    OBJECT_INSTANCE_BEGIN: 140612577298432
                THREAD_ID: 31
                SOCKET_ID: 53
                       IP: ::ffff:127.0.0.1
                     PORT: 55642
                    STATE: ACTIVE
    ...
    ```

+   `PROCESSLIST_DB`

    线程的默认数据库，如果没有选择任何数据库，则为`NULL`。

+   `PROCESSLIST_COMMAND`

    对于前台线程，线程代表客户端执行的命令类型，如果会话空闲，则为`Sleep`。有关线程命令的描述，请参见 Section 10.14, “Examining Server Thread (Process) Information” Information")。此列的值对应于客户端/服务器协议的`COM_*`xxx`*`命令和`Com_*`xxx`*`状态变量。请参见 Section 7.1.10, “Server Status Variables”

    后台线程不代表客户端执行命令，因此此列可能为`NULL`。

+   `PROCESSLIST_TIME`

    线程处于当前状态的秒数。对于复制 SQL 线程，该值是最后一个复制事件的时间戳和复制主机的实际时间之间的秒数。参见 Section 19.2.3, “Replication Threads”。

+   `PROCESSLIST_STATE`

    表示线程正在执行的操作、事件或状态。有关`PROCESSLIST_STATE`值的描述，请参见 Section 10.14, “Examining Server Thread (Process) Information” Information")。如果值为`NULL`，则线程可能对应于空闲客户端会话或其正在执行的工作未使用阶段进行检测。

    大多数状态对应非常快速的操作。如果一个线程在某个状态停留了很多秒，可能存在需要调查的问题。

+   `PROCESSLIST_INFO`

    线程正在执行的语句，如果不执行任何语句，则为`NULL`。该语句可能是发送到服务器的语句，或者如果语句执行其他语句，则是最内部的语句。例如，如果`CALL`语句执行正在执行`SELECT`语句的存储过程，则`PROCESSLIST_INFO`值显示`SELECT`语句。

+   `PARENT_THREAD_ID`

    如果此线程是子线程（由另一个线程生成），则这是生成线程的`THREAD_ID`值。

+   `ROLE`

    未使用。

+   `INSTRUMENTED`

    是否对线程执行的事件进行仪器化。该值为`YES`或`NO`。

    +   对于前台线程，初始的`INSTRUMENTED`值取决于与线程关联的用户帐户是否与`setup_actors`表中的任何行匹配。匹配基于`PROCESSLIST_USER`和`PROCESSLIST_HOST`列的值。

        如果线程生成了子线程，则再次为为子线程创建的`threads`表行进行匹配。

    +   对于后台线程，默认情况下`INSTRUMENTED`为`YES`。不会查询`setup_actors`，因为后台线程没有关联的用户。

    +   对于任何线程，其`INSTRUMENTED`值可以在线程的生命周期内更改。

    要发生对线程执行的事件的监视，必须满足以下条件：

    +   `setup_consumers`表中的`thread_instrumentation`消费者必须为`YES`。

    +   `threads.INSTRUMENTED`列必须为`YES`。

    +   仅对那些在`setup_instruments`表中的`ENABLED`列设置为`YES`的仪器产生的线程事件进行监视。

+   `HISTORY`

    是否记录线程的历史事件。该值为`YES`或`NO`。

    +   对于前台线程，初始的`HISTORY`值取决于与线程关联的用户帐户是否与`setup_actors`表中的任何行匹配。匹配基于`PROCESSLIST_USER`和`PROCESSLIST_HOST`列的值。

        如果线程生成了子线程，则再次为为子线程创建的`threads`表行进行匹配。

    +   对于后台线程，默认情况下`HISTORY`为`YES`。不会查询`setup_actors`，因为后台线程没有关联的用户。

    +   对于任何线程，其`HISTORY`值可以在线程的生命周期内更改。

    要发生对线程的历史事件记录，必须满足以下条件：

    +   必须启用`setup_consumers`表中适当的与历史相关的消费者。例如，在`events_waits_history`和`events_waits_history_long`表中等待事件记录需要相应的`events_waits_history`和`events_waits_history_long`消费者为`YES`。

    +   `threads.HISTORY`列必须为`YES`。

    +   仅对那些在`setup_instruments`表中`ENABLED`列设置为`YES`的仪器产生的线程事件进行记录。

+   `CONNECTION_TYPE`

    用于建立连接的协议，或者对于后台线程为`NULL`。允许的值为`TCP/IP`（未加密建立的 TCP/IP 连接），`SSL/TLS`（加密建立的 TCP/IP 连接），`Socket`（Unix 套接字文件连接），`Named Pipe`（Windows 命名管道连接）和`Shared Memory`（Windows 共享内存连接）。

+   `THREAD_OS_ID`

    如果有的话，由底层操作系统定义的线程或任务标识符：

    +   当 MySQL 线程在其生命周期内与相同的操作系统线程关联时，`THREAD_OS_ID`包含操作系统线程 ID。

    +   当 MySQL 线程在其生命周期内未与相同的操作系统线程关联时，`THREAD_OS_ID`包含`NULL`。这在使用线程池插件时（参见 Section 7.6.3, “MySQL Enterprise Thread Pool”）的用户会话中很常见。

    对于 Windows，`THREAD_OS_ID`对应于在 Process Explorer 中可见的线程 ID（[`technet.microsoft.com/en-us/sysinternals/bb896653.aspx`](https://technet.microsoft.com/en-us/sysinternals/bb896653.aspx)）。

    对于 Linux，`THREAD_OS_ID`对应于`gettid()`函数的值。例如，可以使用**perf**或**ps -L**命令，或在`proc`文件系统（`/proc/*`[pid]`*/task/*`[tid]`*`）中公开此值。有关更多信息，请参阅`perf-stat(1)`，`ps(1)`和`proc(5)`手册页。

+   `RESOURCE_GROUP`

    资源组标签。如果当前平台或服务器配置不支持资源组，则此值为`NULL`（请参阅 Resource Group Restrictions）。

+   `EXECUTION_ENGINE`

    查询执行引擎。值为 `PRIMARY` 或 `SECONDARY`。用于 MySQL HeatWave 服务和 HeatWave，其中 `PRIMARY` 引擎是 `InnoDB`，`SECONDARY` 引擎是 HeatWave (`RAPID`)。对于 MySQL 社区版服务器、MySQL 企业版服务器（本地）和没有 HeatWave 的 MySQL HeatWave 服务，值始终为 `PRIMARY`。此列在 MySQL 8.0.29 版本中添加。

+   `CONTROLLED_MEMORY`

    线程使用的受控内存量。

    此列在 MySQL 8.0.31 版本中添加。

+   `MAX_CONTROLLED_MEMORY`

    线程执行期间看到的 `CONTROLLED_MEMORY` 的最大值。

    此列在 MySQL 8.0.31 版本中添加。

+   `TOTAL_MEMORY`

    线程使用的当前内存量，无论是否受控。

    此列在 MySQL 8.0.31 版本中添加。

+   `MAX_TOTAL_MEMORY`

    线程执行期间看到的 `TOTAL_MEMORY` 的最大值。

    此列在 MySQL 8.0.31 版本中添加。

+   `TELEMETRY_ACTIVE`

    线程是否附加了活动的遥测会话。值为 `YES` 或 `NO`。

    此列在 MySQL 8.0.33 版本中添加。

`threads` 表具有以下索引：

+   在 (`THREAD_ID`) 上的主键

+   在 (`NAME`) 上的索引

+   在 (`PROCESSLIST_ID`) 上的索引

+   在 (`PROCESSLIST_USER`, `PROCESSLIST_HOST`) 上的索引

+   在 (`PROCESSLIST_HOST`) 上的索引

+   在 (`THREAD_OS_ID`) 上的索引

+   在 (`RESOURCE_GROUP`) 上的索引

`TRUNCATE TABLE` 不允许用于 `threads` 表。
