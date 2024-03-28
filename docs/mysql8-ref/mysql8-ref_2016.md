> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-instruments-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-instruments-table.html)

#### 29.12.2.3 setup_instruments 表

`setup_instruments` 表列出了可以收集事件的被检测对象的类别：

```sql
mysql> SELECT * FROM performance_schema.setup_instruments\G
*************************** 1\. row ***************************
         NAME: wait/synch/mutex/pfs/LOCK_pfs_share_list
      ENABLED: NO
        TIMED: NO
   PROPERTIES: singleton
        FLAGS: NULL
   VOLATILITY: 1
DOCUMENTATION: Components can provide their own performance_schema tables. 
This lock protects the list of such tables definitions.
...
*************************** 410\. row ***************************
         NAME: stage/sql/executing
      ENABLED: NO
        TIMED: NO
   PROPERTIES:
        FLAGS: NULL
   VOLATILITY: 0
DOCUMENTATION: NULL
...
*************************** 733\. row ***************************
         NAME: statement/abstract/Query
      ENABLED: YES
        TIMED: YES
   PROPERTIES: mutable
        FLAGS: NULL
   VOLATILITY: 0
DOCUMENTATION: SQL query just received from the network. 
At this point, the real statement type is unknown, the type 
will be refined after SQL parsing.
...
*************************** 737\. row ***************************
         NAME: memory/performance_schema/mutex_instances
      ENABLED: YES
        TIMED: NULL
   PROPERTIES: global_statistics
        FLAGS:
   VOLATILITY: 1
DOCUMENTATION: Memory used for table performance_schema.mutex_instances
...
*************************** 823\. row ***************************
         NAME: memory/sql/Prepared_statement::infrastructure
      ENABLED: YES
        TIMED: NULL
   PROPERTIES: controlled_by_default
        FLAGS: controlled
   VOLATILITY: 0
DOCUMENTATION: Map infrastructure for prepared statements per session.
...
```

源代码中添加的每个仪器都为 `setup_instruments` 表提供一行，即使未执行仪器化代码。当启用并执行仪器时，将创建仪器化实例，这些实例在 `*`xxx`*_instances` 表中可见，例如 `file_instances` 或 `rwlock_instances`。

大多数 `setup_instruments` 行的修改立即影响监视。对于某些仪器，修改仅在服务器启动时生效；在运行时更改它们没有影响。这主要影响服务器中的互斥体、条件和读写锁，尽管可能还有其他仪器也是如此。

欲了解有关 `setup_instruments` 表在事件过滤中的作用的更多信息，请参阅 Section 29.4.3, “Event Pre-Filtering”。

`setup_instruments` 表包含以下列：

+   `NAME`

    仪器名称。仪器名称可能有多个部分并形成层次结构，如 Section 29.6, “Performance Schema Instrument Naming Conventions” 中所讨论的。从执行仪器产生的事件具有从仪器 `NAME` 值中获取的 `EVENT_NAME` 值。（事件实际上没有“名称”，但这提供了一种将事件与仪器关联的方式。）

+   `ENABLED`

    仪器是否启用。值为 `YES` 或 `NO`。禁用的仪器不会产生事件。此列可修改，尽管对于已创建的仪器设置 `ENABLED` 没有影响。

+   `TIMED`

    仪器是否计时。值为 `YES`、`NO` 或 `NULL`。此列可修改，尽管对于已创建的仪器设置 `TIMED` 没有影响。

    `TIMED` 值为 `NULL` 表示该仪器不支持计时。例如，内存操作不计时，因此其 `TIMED` 列为 `NULL`。

    为支持计时的仪器设置`TIMED`为`NULL`没有效果，为不支持计时的仪器设置`TIMED`为非`NULL`也没有效果。

    如果启用的仪器没有计时，那么仪器代码是启用的，但计时器不是。由仪器产生的事件的`TIMER_START`、`TIMER_END`和`TIMER_WAIT`计时器值为`NULL`。这反过来导致在汇总表中计算总和、最小值、最大值和平均时间值时忽略这些值。

+   `PROPERTIES`

    仪器属性。此列使用`SET`数据类型，因此每个仪器可以设置以下列表中的多个标志：

    +   `controlled_by_default`: 默认情况下为该仪器收集内存。

    +   `global_statistics`: 该仪器仅生成全局摘要。较细级别的摘要不可用，例如每个线程、账户、用户或主机。例如，大多数内存仪器仅生成全局摘要。

    +   `mutable`: 该仪器可以“变异”为更具体的仪器。此属性仅适用于语句仪器。

    +   `progress`: 该仪器能够报告进度数据。此属性仅适用于阶段仪器。

    +   `singleton`: 该仪器具有单个实例。例如，服务器中的大多数全局互斥锁都是单例的，因此相应的仪器也是如此。

    +   `user`: 该仪器与用户工作量直接相关（与系统工作量相反）。其中一个这样的仪器是`wait/io/socket/sql/client_connection`。

+   `FLAGS`

    仪器的内存是否受控。

    该标志仅支持非全局内存仪器，并且可以设置或取消设置。例如：

    ```sql
     SQL> UPDATE PERFORMANCE_SCHEMA.SETUP_INTRUMENTS SET FLAGS="controlled" WHERE NAME='memory/sql/NET::buff';
    ```

    注意

    尝试在非内存仪器或全局内存仪器上设置`FLAGS = controlled`会静默失败。

+   `VOLATILITY`

    仪器的不稳定性。不稳定性值从低到高。这些值对应于`mysql/psi/psi_base.h`头文件中定义的`PSI_VOLATILITY_*`xxx`*`常量：

    ```sql
    #define PSI_VOLATILITY_UNKNOWN 0
    #define PSI_VOLATILITY_PERMANENT 1
    #define PSI_VOLATILITY_PROVISIONING 2
    #define PSI_VOLATILITY_DDL 3
    #define PSI_VOLATILITY_CACHE 4
    #define PSI_VOLATILITY_SESSION 5
    #define PSI_VOLATILITY_TRANSACTION 6
    #define PSI_VOLATILITY_QUERY 7
    #define PSI_VOLATILITY_INTRA_QUERY 8
    ```

    `VOLATILITY`列纯粹是信息性的，为用户（以及性能模式代码）提供有关仪器运行时行为的一些提示。

    具有低不稳定性指数（PERMANENT = 1）的仪器在服务器启动时创建一次，并且在正常服务器操作期间永远不会被销毁或重新创建。它们仅在服务器关闭时被销毁。

    例如，`wait/synch/mutex/pfs/LOCK_pfs_share_list`互斥锁被定义为具有不变性为 1，这意味着它只创建一次。仪器本身可能产生的开销（即互斥锁初始化）对于这个仪器没有影响。运行时开销仅在锁定或解锁互斥锁时发生。

    具有较高不稳定性指数（例如，SESSION = 5）的仪器为每个用户会话创建和销毁。例如，`wait/synch/mutex/sql/THD::LOCK_query_plan`互斥锁在每次会话连接时创建，并在会话断开时销毁。

    这个互斥体对性能模式的开销更为敏感，因为开销不仅来自于锁定和解锁的仪器化，还来自于更频繁执行的互斥体创建和销毁的仪器化。

    另一个不稳定性的方面涉及`ENABLED`列的更新实际上是否产生某种效果：

    +   对`ENABLED`的更新会影响随后创建的被仪器化对象，但不会影响已经创建的仪器。

    +   更“不稳定”的仪器会更快地使用`setup_instruments`表中的新设置。

    例如，此语句不会影响现有会话的`LOCK_query_plan`互斥体，但会影响在更新后创建的新会话：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED=*value*
    WHERE NAME = 'wait/synch/mutex/sql/THD::LOCK_query_plan';
    ```

    这个语句实际上根本没有任何效果：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED=*value*
    WHERE NAME = 'wait/synch/mutex/pfs/LOCK_pfs_share_list';
    ```

    这个互斥体是永久的，在更新执行之前已经创建。互斥体永远不会再次创建，因此`setup_instruments`中的`ENABLED`值永远不会被使用。要启用或禁用此互斥体，请改用`mutex_instances`表。

+   `DOCUMENTATION`

    描述仪器目的的字符串。如果没有可用的描述，则值为`NULL`。

`setup_instruments`表具有以下索引：

+   主键为(`NAME`)

不允许对`setup_instruments`表进行`TRUNCATE TABLE`。

截至 MySQL 8.0.27，为了辅助监控和故障排除，性能模式仪器化用于将被仪器化线程的名称导出到操作系统。这使得能够显示线程名称的实用程序（如调试器和 Unix **ps**命令）能够显示不同的**mysqld**线程名称，而不是“mysqld”。此功能仅在 Linux、macOS 和 Windows 上受支持。

假设**mysqld**正在运行在一个支持此调用语法的**ps**版本的系统上：

```sql
ps -C mysqld H -o "pid tid cmd comm"
```

在不将线程名称导出到操作系统的情况下，该命令显示类似于这样的输出，其中大多数`COMMAND`值为`mysqld`：

```sql
 PID   TID CMD                         COMMAND
 1377  1377 /usr/sbin/mysqld            mysqld
 1377  1528 /usr/sbin/mysqld            mysqld
 1377  1529 /usr/sbin/mysqld            mysqld
 1377  1530 /usr/sbin/mysqld            mysqld
 1377  1531 /usr/sbin/mysqld            mysqld
 1377  1534 /usr/sbin/mysqld            mysqld
 1377  1535 /usr/sbin/mysqld            mysqld
 1377  1588 /usr/sbin/mysqld            xpl_worker1
 1377  1589 /usr/sbin/mysqld            xpl_worker0
 1377  1590 /usr/sbin/mysqld            mysqld
 1377  1594 /usr/sbin/mysqld            mysqld
 1377  1595 /usr/sbin/mysqld            mysqld
```

将线程名称导出到操作系统后，输出看起来像这样，线程的名称类似于它们的仪器名称：

```sql
 PID   TID CMD                         COMMAND
27668 27668 /usr/sbin/mysqld            mysqld
27668 27671 /usr/sbin/mysqld            ib_io_ibuf
27668 27672 /usr/sbin/mysqld            ib_io_log
27668 27673 /usr/sbin/mysqld            ib_io_rd-1
27668 27674 /usr/sbin/mysqld            ib_io_rd-2
27668 27677 /usr/sbin/mysqld            ib_io_wr-1
27668 27678 /usr/sbin/mysqld            ib_io_wr-2
27668 27699 /usr/sbin/mysqld            xpl_worker-2
27668 27700 /usr/sbin/mysqld            xpl_accept-1
27668 27710 /usr/sbin/mysqld            evt_sched
27668 27711 /usr/sbin/mysqld            sig_handler
27668 27933 /usr/sbin/mysqld            connection
```

同一类中的不同线程实例被编号以提供可行的不同名称。由于名称长度受到可能大量连接的限制，连接被简单命名为`connection`。
