> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-events-waits-current-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-waits-current-table.html)

#### 29.12.4.1 events_waits_current 表

`events_waits_current` 表包含当前的等待事件。该表为每个线程存储一行，显示线程最近监视等待事件的当前状态，因此没有用于配置表大小的系统变量。

在包含等待事件行的表中，`events_waits_current` 是最基本的。其他包含等待事件行的表从当前事件中逻辑派生。例如，`events_waits_history` 和 `events_waits_history_long` 表是最近已结束的等待事件的集合，每个线程和全局跨所有线程的最大行数。

有关三个等待事件表之间关系的更多信息，请参见 Section 29.9, “Performance Schema Tables for Current and Historical Events”。

有关配置是否收集等待事件的信息，请参见 Section 29.12.4, “Performance Schema Wait Event Tables”。

`events_waits_current` 表具有以下列：

+   `THREAD_ID`, `EVENT_ID`

    与事件相关联的线程以及事件开始时的线程当前事件编号。`THREAD_ID` 和 `EVENT_ID` 值一起唯一标识行。没有两行具有相同的值对。

+   `END_EVENT_ID`

    当事件开始时，此列设置为 `NULL`，并在事件结束时更新为线程当前事件编号。

+   `EVENT_NAME`

    产生事件的工具的名称。这是来自 `setup_instruments` 表的 `NAME` 值。工具名称可能有多个部分并形成层次结构，如 Section 29.6, “Performance Schema Instrument Naming Conventions” 中所讨论的。

+   `SOURCE`

    包含产生事件的有仪器代码的源文件的名称和仪器发生的文件中的行号。这使您可以检查源代码以确定涉及的确切代码。例如，如果互斥锁或锁被阻塞，您可以检查发生这种情况的上下文。

+   `TIMER_START`，`TIMER_END`，`TIMER_WAIT`

    事件的时间信息。这些值的单位是皮秒（万亿分之一秒）。`TIMER_START`和`TIMER_END`值表示事件计时开始和结束的时间。`TIMER_WAIT`是事件经过的时间（持续时间）。

    如果事件尚未完成，`TIMER_END`是当前计时器值，`TIMER_WAIT`是到目前为止经过的时间（`TIMER_END` − `TIMER_START`）。

    如果事件来自具有`TIMED = NO`的工具，将不会收集时间信息，`TIMER_START`，`TIMER_END`和`TIMER_WAIT`都为`NULL`。

    有关以皮秒为事件时间单位和影响时间值的因素的讨论，请参阅 Section 29.4.1, “Performance Schema Event Timing”。

+   `SPINS`

    对于互斥锁，自旋轮数。如果值为`NULL`，则代码不使用自旋轮数或自旋未被记录。

+   `OBJECT_SCHEMA`，`OBJECT_NAME`，`OBJECT_TYPE`，`OBJECT_INSTANCE_BEGIN`

    这些列标识了“被操作的”对象。具体含义取决于对象类型。

    对于一个同步对象（`cond`，`mutex`，`rwlock`）：

    +   `OBJECT_SCHEMA`，`OBJECT_NAME`和`OBJECT_TYPE`为`NULL`。

    +   `OBJECT_INSTANCE_BEGIN`是内存中同步对象的地址。

    对于文件 I/O 对象：

    +   `OBJECT_SCHEMA`为`NULL`。

    +   `OBJECT_NAME`是文件名。

    +   `OBJECT_TYPE`为`FILE`。

    +   `OBJECT_INSTANCE_BEGIN`是内存中的地址。

    对于套接字对象：

    +   `OBJECT_NAME`是套接字的`IP:PORT`值。

    +   `OBJECT_INSTANCE_BEGIN`是内存中的地址。

    对于表 I/O 对象：

    +   `OBJECT_SCHEMA`是包含表的模式的名称。

    +   `OBJECT_NAME`是表名。

    +   对于持久基表的`OBJECT_TYPE`为`TABLE`或临时表的`OBJECT_TYPE`为`TEMPORARY TABLE`。

    +   `OBJECT_INSTANCE_BEGIN`是内存中的地址。

    `OBJECT_INSTANCE_BEGIN`值本身没有意义，除非不同的值表示不同的对象。`OBJECT_INSTANCE_BEGIN`可用于调试。例如，可以与`GROUP BY OBJECT_INSTANCE_BEGIN`一起使用，以查看对 1,000 个互斥锁（保护 1,000 个页面或数据块）的负载是否均匀分布或只是击中了一些瓶颈。如果在日志文件或其他调试或性能工具中看到相同的对象地址，这可以帮助您与其他信息源进行关联。

+   `INDEX_NAME`

    使用的索引名称。`PRIMARY`表示表的主索引。`NULL`表示未使用索引。

+   `NESTING_EVENT_ID`

    此事件嵌套在其中的事件的`EVENT_ID`值。

+   `NESTING_EVENT_TYPE`

    嵌套事件类型。值为`TRANSACTION`，`STATEMENT`，`STAGE`或`WAIT`。

+   `OPERATION`

    执行的操作类型，例如`lock`，`read`或`write`。

+   `NUMBER_OF_BYTES`

    操作读取或写入的字节数。对于表 I/O 等待（`wait/io/table/sql/handler`工具的事件），`NUMBER_OF_BYTES`表示行数。如果值大于 1，则该事件是批量 I/O 操作。以下讨论描述了独占单行报告和反映批量 I/O 的报告之间的区别。

    MySQL 使用嵌套循环实现连接。性能模式仪表化的工作是为连接中的每个表提供行数和累积执行时间。假设执行以下形式的连接查询，使用表连接顺序为`t1`，`t2`，`t3`：

    ```sql
    SELECT ... FROM t1 JOIN t2 ON ... JOIN t3 ON ...
    ```

    表“扇出”是在连接处理过程中添加表后行数增加或减少的情况。如果表`t3`的扇出大于 1，则大部分行提取操作是针对该表的。假设连接从`t1`提取 10 行，从`t2`每行`t1`提取 20 行，从`t3`每行`t2`提取 30 行。使用单行报告，总仪表化操作数为：

    ```sql
    10 + (10 * 20) + (10 * 20 * 30) = 6210
    ```

    通过按扫描（即从`t1`和`t2`的唯一组合）对其进行聚合，可以显著减少仪表化操作的数量。使用批量 I/O 报告，性能模式为内部表`t3`的每次扫描产生一个事件，而不是每行，仪表化行操作数量减少为：

    ```sql
    10 + (10 * 20) + (10 * 20) = 410
    ```

    这是 93%的减少，说明批量报告策略通过减少报告调用的数量显著减少了表 I/O 的性能模式开销。权衡是事件定时的准确性较低。与每行报告中的单行操作的时间不同，批量 I/O 的时间包括用于操作的时间，例如连接缓冲，聚合和将行返回给客户端的时间。

    要发生批量 I/O 报告，必须满足以下条件：

    +   查询执行访问查询块的最内部表（对于单表查询，该表计为最内部）

    +   查询执行不会从表中请求单行（因此，例如，`eq_ref`访问会阻止批量报告的使用）

    +   查询执行不会评估包含表访问的子查询的表

+   `FLAGS`

    保留供将来使用。

`events_waits_current`表具有以下索引：

+   主键为（`THREAD_ID`，`EVENT_ID`）

允许对`events_waits_current`表进行`TRUNCATE TABLE`。它会删除行。
