# 29.6 性能模式仪器命名约定

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-instrument-naming.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-instrument-naming.html)

一个仪器名称由一系列由 `'/'` 字符分隔的元素组成。示例名称：

```sql
wait/io/file/myisam/log
wait/io/file/mysys/charset
wait/lock/table/sql/handler
wait/synch/cond/mysys/COND_alarm
wait/synch/cond/sql/BINLOG::update_cond
wait/synch/mutex/mysys/BITMAP_mutex
wait/synch/mutex/sql/LOCK_delete
wait/synch/rwlock/sql/Query_cache_query::lock
stage/sql/closing tables
stage/sql/Sorting result
statement/com/Execute
statement/com/Query
statement/sql/create_table
statement/sql/lock_tables
errors
```

仪器名称空间具有类似树状结构。仪器名称中的元素从左到右提供了从更一般到更具体的进展。名称中的元素数量取决于仪器的类型。

在名称中给定元素的解释取决于其左侧的元素。例如，`myisam` 出现在以下两个名称中，但第一个名称中的 `myisam` 与文件 I/O 有关，而第二个名称中的 `myisam` 与同步仪器有关：

```sql
wait/io/file/myisam/log
wait/synch/cond/myisam/MI_SORT_INFO::cond
```

仪器名称由性能模式实现定义的前缀结构和由实现仪器代码的开发人员定义的后缀组成。仪器前缀的顶层元素指示仪器的类型。此元素还确定`performance_timers` 表中的哪个事件计时器适用于该仪器。对于仪器名称的前缀部分，顶层元素指示仪器的类型。

仪器名称的后缀部分来自仪器本身的代码。后缀可能包括以下级别：

+   主要元素的名称（服务器模块，如 `myisam`，`innodb`，`mysys` 或 `sql`）或插件名称。

+   代码中变量的名称，形式为 *`XXX`*（全局变量）或 `*`CCC`*::*`MMM`*`（类 *`CCC`* 中的成员 *`MMM`*）。示例：`COND_thread_cache`，`THR_LOCK_myisam`，`BINLOG::LOCK_index`。

+   顶层仪器元素

+   空闲仪器元素

+   错误仪器元素

+   内存仪器元素

+   阶段仪器元素

+   语句仪器元素

+   线程仪器元素

+   等待仪器元素

### 顶层仪器元素

+   `idle`: 一个被检测的空闲事件。这个仪器没有进一步的元素。

+   `error`: 一个被检测的错误事件。这个仪器没有进一步的元素。

+   `memory`: 一个被检测的内存事件。

+   `stage`: 一个被检测的阶段事件。

+   `statement`: 一个被检测的语句事件。

+   `transaction`: 一个被检测的事务事件。这个仪器没有进一步的元素。

+   `wait`: 一个被检测的等待事件。

### 空闲仪器元素

`idle`仪器用于空闲事件，性能模式生成这些事件，如第 29.12.3.5 节中`socket_instances.STATE`列的描述所述。

### 错误仪器元素

`error`仪器指示是否收集有关服务器错误和警告的信息。这个仪器默认启用。`setup_instruments`表中`error`行的`TIMED`列不适用，因为不收集时间信息。

### 内存仪器元素

内存仪器默认情况下是启用的。内存仪器可以在启动时启用或禁用，或者通过更新`setup_instruments`表中相关仪器的`ENABLED`列来在运行时动态启用或禁用。内存仪器的名称形式为`memory/*`code_area`*/*`instrument_name`*`，其中*`code_area`*是一个值，例如`sql`或`myisam`，而*`instrument_name`*是仪器的详细信息。

以`memory/performance_schema/`前缀命名的仪器公开了性能模式中内部缓冲区分配了多少内存。`memory/performance_schema/`仪器是内置的，始终启用，并且无法在启动时或运行时禁用。内置内存仪器仅在`memory_summary_global_by_event_name`表中显示。有关更多信息，请参见第 29.17 节，“性能模式内存分配模型”。

### 阶段仪器元素

阶段工具的名称形式为`stage/*`code_area`*/*`stage_name`*`，其中*`code_area`*是诸如`sql`或`myisam`之类的值，*`stage_name`*表示语句处理阶段，例如`Sorting result`或`Sending data`。阶段对应于`SHOW PROCESSLIST`显示的线程状态或在信息模式`PROCESSLIST`表中可见的状态。

### 语句工具元素

+   `statement/abstract/*`：用于语句操作的抽象工具。在确切语句类型未知的语句分类早期阶段使用抽象工具，然后在了解类型后将其更改为更具体的语句工具。有关此过程的描述，请参阅 Section 29.12.6, “Performance Schema Statement Event Tables”。

+   `statement/com`：一个被检测的命令操作。这些名称对应于`COM_*`xxx`*`操作（请参阅`mysql_com.h`头文件和`sql/sql_parse.cc`。例如，`statement/com/Connect`和`statement/com/Init DB`工具对应于`COM_CONNECT`和`COM_INIT_DB`命令。

+   `statement/scheduler/event`：用于跟踪事件调度器执行的所有事件的单个工具。当计划事件开始执行时，此工具开始发挥作用。

+   `statement/sp`：由存储程序执行的一个被检测的内部指令。例如，`statement/sp/cfetch`和`statement/sp/freturn`工具用于游标获取和函数返回指令。

+   `statement/sql`：一个被检测的 SQL 语句操作。例如，`statement/sql/create_db`和`statement/sql/select`工具用于`CREATE DATABASE`和`SELECT`语句。

### 线程工具元素

被检测的线程显示在`setup_threads`表中，该表公开线程类名称和属性。

线程工具以`thread`开头（例如，`thread/sql/parser_service`或`thread/performance_schema/setup`）。

`ndbcluster`插件线程的线程工具名称以`thread/ndbcluster/`开头；有关更多信息，请参阅 ndbcluster Plugin Threads。

### 等待工具元素

+   `wait/io`

    一个被检测的 I/O 操作。

    +   `wait/io/file`

        一个被检测的文件 I/O 操作。对于文件，等待是等待文件操作完成的时间（例如，调用`fwrite()`）。由于缓存，磁盘上的物理文件 I/O 可能不会在此调用中发生。

    +   `wait/io/socket`

        一个被检测的套接字操作。套接字工具的名称形式为`wait/io/socket/sql/*`socket_type`*`。服务器为它支持的每种网络协议都有一个监听套接字。与 TCP/IP 或 Unix 套接字文件连接的监听套接字相关的工具具有`server_tcpip_socket`或`server_unix_socket`的*`socket_type`*值。当一个监听套接字检测到一个连接时，服务器将连接传输给由单独线程管理的新套接字。新连接线程的工具具有`client_connection`的*`socket_type`*值。

    +   `wait/io/table`

        一个被检测的表 I/O 操作。这些包括对持久基表或临时表的行级访问。影响行的操作包括获取、插入、更新和删除。对于视图，等待与视图引用的基表相关联。

        与大多数等待不同，表 I/O 等待可能包括其他等待。例如，表 I/O 可能包括文件 I/O 或内存操作。因此，表 I/O 等待的`events_waits_current`通常有两行。有关更多信息，请参见第 29.8 节，“性能模式原子和分子事件”。

        一些行操作可能导致多个表 I/O 等待。例如，插入可能激活触发器导致更新。

+   `wait/lock`

    一个被检测的锁操作。

    +   `wait/lock/table`

        一个被检测的表锁操作。

    +   `wait/lock/metadata/sql/mdl`

        一个被检测的元数据锁操作。

+   `wait/synch`

    一个被检测的同步对象。对于同步对象，`TIMER_WAIT`时间包括在尝试获取对象锁时被阻塞的时间。

    +   `wait/synch/cond`

        一个条件被一个线程用来通知其他线程他们等待的事情已经发生。如果一个线程在等待一个条件，它可以唤醒并继续执行。如果有多个线程在等待，它们都可以被唤醒并竞争它们等待的资源。

    +   `wait/synch/mutex`

        一个用于允许访问资源（如可执行代码段）并防止其他线程访问资源的互斥对象。

    +   `wait/synch/prlock`

        一个优先级读/写锁锁对象。

    +   `wait/synch/rwlock`

        用于锁定特定变量以防止其他线程使用的普通读/写锁对象。多个线程可以同时获取共享读锁。独占写锁一次只能被一个线程获取。

    +   `wait/synch/sxlock`

        一个共享-独占（SX）锁是一种类型的 rwlock 锁对象，它允许其他线程进行不一致的读取，同时提供对共享资源的写访问。`sxlocks`优化并发性，提高读写工作负载的可伸缩性。
