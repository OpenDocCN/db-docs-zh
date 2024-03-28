> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-summary-tables.html)

#### 29.12.20.10 内存摘要表

性能模式仪表盘记录内存使用情况并汇总内存使用统计信息，详细说明以下因素：

+   使用的内存类型（各种缓存，内部缓冲区等）

+   执行内存操作的线程，帐户，用户，主机

性能模式仪表盘记录了内存使用的以下方面。

+   使用的内存大小

+   操作计数

+   低水位和高水位

内存大小有助于理解或调整服务器的内存消耗。

操作计数有助于理解或调整服务器对内存分配器施加的整体压力，这对性能有影响。一百万次分配一个字节不同于一次分配一百万字节；跟踪两种大小和计数可以揭示差异。

低水位和高水位对于检测工作负载峰值、整体���作负载稳定性和可能的内存泄漏至关重要。

内存摘要表不包含时间信息，因为内存事件没有时间。

有关收集内存使用数据的信息，请参阅[内存仪表行为](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-summary-tables.html#memory-instrumentation-behavior)。

示例内存事件摘要信息：

```sql
mysql> SELECT *
       FROM performance_schema.memory_summary_global_by_event_name
       WHERE EVENT_NAME = 'memory/sql/TABLE'\G
*************************** 1\. row ***************************
                  EVENT_NAME: memory/sql/TABLE
                 COUNT_ALLOC: 1381
                  COUNT_FREE: 924
   SUM_NUMBER_OF_BYTES_ALLOC: 2059873
    SUM_NUMBER_OF_BYTES_FREE: 1407432
              LOW_COUNT_USED: 0
          CURRENT_COUNT_USED: 457
             HIGH_COUNT_USED: 461
    LOW_NUMBER_OF_BYTES_USED: 0
CURRENT_NUMBER_OF_BYTES_USED: 652441
   HIGH_NUMBER_OF_BYTES_USED: 669269
```

每个内存摘要表都有一个或多个分组列，指示表如何聚合事件。事件名称指的是[`setup_instruments`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-instruments-table.html)表中事件仪表的名称：

+   [`按帐户按事件名称汇总的内存摘要`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-summary-tables.html)具有`USER`，`HOST`和`EVENT_NAME`列。每行总结了给定帐户（用户和主机组合）和事件名称的事件。

+   [`按主机按事件名称汇总的内存摘要`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-summary-tables.html)具有`HOST`和`EVENT_NAME`列。每行总结了给定主机和事件名称的事件。

+   [`按线程按事件名称汇总的内存摘要`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-summary-tables.html)具有`THREAD_ID`和`EVENT_NAME`列。每行总结了给定线程和事件名称的事件。

+   [`按用户按事件名称汇总的内存摘要`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-summary-tables.html)具有`USER`和`EVENT_NAME`列。每行总结了给定用户和事件名称的事件。

+   [`按事件名称全局汇总的内存摘要`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-summary-tables.html)具有`EVENT_NAME`列。每行总结了给定事件名称的事件。

每个内存摘要表都有这些包含聚合值的摘要列：

+   `COUNT_ALLOC`，`COUNT_FREE`

    内存分配和内存释放函数调用次数的聚合数。

+   `SUM_NUMBER_OF_BYTES_ALLOC`，`SUM_NUMBER_OF_BYTES_FREE`

    已分配和释放内存块的聚合大小。

+   `CURRENT_COUNT_USED`

    尚未释放的当前分配块的聚合数量。这是一个便利列，等于 `COUNT_ALLOC` - `COUNT_FREE`。

+   `CURRENT_NUMBER_OF_BYTES_USED`

    尚未释放的当前分配内存块的聚合大小。这是一个便利列，等于 `SUM_NUMBER_OF_BYTES_ALLOC` - `SUM_NUMBER_OF_BYTES_FREE`。

+   `LOW_COUNT_USED`，`HIGH_COUNT_USED`

    与`CURRENT_COUNT_USED`列对应的低水位和高水位标记。

+   `LOW_NUMBER_OF_BYTES_USED`，`HIGH_NUMBER_OF_BYTES_USED`

    与`CURRENT_NUMBER_OF_BYTES_USED`列对应的低水位和高水位标记。

内存摘要表具有以下索引：

+   `memory_summary_by_account_by_event_name`:

    +   主键为(`USER`, `HOST`, `EVENT_NAME`)

+   `memory_summary_by_host_by_event_name`:

    +   主键为(`HOST`, `EVENT_NAME`)

+   `memory_summary_by_thread_by_event_name`:

    +   主键为(`THREAD_ID`, `EVENT_NAME`)

+   `memory_summary_by_user_by_event_name`:

    +   主键为(`USER`, `EVENT_NAME`)

+   `memory_summary_global_by_event_name`:

    +   主键为(`EVENT_NAME`)

`TRUNCATE TABLE` 允许用于内存摘要表。其效果如下：

+   一般来说，截断会重置统计数据的基线，但不会改变服务器状态。也就是说，截断内存表不会释放内存。

+   `COUNT_ALLOC` 和 `COUNT_FREE` 通过减少相同值来重置为新基线。

+   同样，`SUM_NUMBER_OF_BYTES_ALLOC` 和 `SUM_NUMBER_OF_BYTES_FREE` 重置为新基线。

+   `LOW_COUNT_USED` 和 `HIGH_COUNT_USED` 重置为 `CURRENT_COUNT_USED`。

+   `LOW_NUMBER_OF_BYTES_USED` 和 `HIGH_NUMBER_OF_BYTES_USED` 重置为 `CURRENT_NUMBER_OF_BYTES_USED`。

此外，每个按帐户、主机、用户或线程聚合的内存摘要表在其依赖的连接表被截断或`memory_summary_global_by_event_name` 被截断时会被隐式截断。详情请参见第 29.12.8 节，“性能模式连接表”。

##### 内存工具行为

内存工具在`setup_instruments`表中列出，并具有形式为`memory/*`code_area`*/*`instrument_name`*`的名称。内存工具默认启用。

以`memory/performance_schema/`前缀命名的工具公开了性能模式内部缓冲区分配了多少内存。`memory/performance_schema/`工具是内置的，始终启用，并且无法在启动时或运行时禁用。内置内存工具仅在`memory_summary_global_by_event_name`表中显示。

要在服务器启动时控制内存工具状态，请在您的`my.cnf`文件中使用以下行：

+   启用：

    ```sql
    [mysqld]
    performance-schema-instrument='memory/%=ON'
    ```

+   禁用：

    ```sql
    [mysqld]
    performance-schema-instrument='memory/%=OFF'
    ```

要在运行时控制内存工具状态，请更新`setup_instruments`表中相关工具的`ENABLED`列：

+   启用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'YES'
    WHERE NAME LIKE 'memory/%';
    ```

+   禁用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'NO'
    WHERE NAME LIKE 'memory/%';
    ```

对于内存工具，`setup_instruments`中的`TIMED`列被忽略，因为内存操作不计时。

当服务器中的线程执行已被工具化的内存分配时，适用以下规则：

+   如果线程未被工具化或内存工具未启用，则分配的内存块不会被工具化。

+   否则（即线程和工具均已启用），分配的内存块将被工具化。

对于释放，适用以下规则：

+   如果内存分配操作已被工具化，则无论当前工具或线程启用状态如何，相应的释放操作都会被工具化。

+   如果内存分配操作未被工具化，则无论当前工具或线程启用状态如何，相应的释放操作都不会被工具化。

对于每个线程的统计信息，适用以下规则。

当分配大小为*`N`*的工具化内存块时，性能模式将对内存摘要表列进行以下更新：

+   `COUNT_ALLOC`：增加 1

+   `CURRENT_COUNT_USED`：增加 1

+   `HIGH_COUNT_USED`：如果`CURRENT_COUNT_USED`是新的最大值，则增加

+   `SUM_NUMBER_OF_BYTES_ALLOC`：增加*`N`*

+   `CURRENT_NUMBER_OF_BYTES_USED`：增加*`N`*

+   `HIGH_NUMBER_OF_BYTES_USED`：如果`CURRENT_NUMBER_OF_BYTES_USED`是新的最大值，则增加

当工具化的内存块被释放时，性能模式将对内存摘要表列进行以下更新：

+   `COUNT_FREE`：增加 1

+   `CURRENT_COUNT_USED`：减少 1

+   `LOW_COUNT_USED`：如果`CURRENT_COUNT_USED`是新的最小值，则减少

+   `SUM_NUMBER_OF_BYTES_FREE`：增加*`N`*

+   `CURRENT_NUMBER_OF_BYTES_USED`：减少*`N`*

+   `LOW_NUMBER_OF_BYTES_USED`：如果`CURRENT_NUMBER_OF_BYTES_USED`是新的最小值，则减少

对于更高级别的聚合（全局、按账户、按用户、按主机），低水位和高水位的规则与预期相同。

+   `LOW_COUNT_USED` 和 `LOW_NUMBER_OF_BYTES_USED` 是较低的估计值。性能模式报告的值保证小于或等于运行时实际使用的内存的最低计数或大小。

+   `HIGH_COUNT_USED` 和 `HIGH_NUMBER_OF_BYTES_USED` 是较高的估计值。性能模式报告的值保证大于或等于运行时实际使用的内存的最高计数或大小。

对于除`memory_summary_global_by_event_name`之外的摘要表中的较低估计值，如果内存所有权在线程之间转移，则值可能为负。

这里是一个估算计算的示例；但请注意，估算实现可能会发生变化：

线程 1 在执行过程中使用的内存范围为 1MB 到 2MB，如`memory_summary_by_thread_by_event_name`表的 `LOW_NUMBER_OF_BYTES_USED` 和 `HIGH_NUMBER_OF_BYTES_USED` 列所报告的那样。

线程 2 在执行过程中使用的内存范围为 10MB 到 12MB，如同报告的那样。

当这两个线程属于同一用户账户时，每个账户摘要估计该账户在 11MB 到 14MB 的内存范围内。也就是说，较高级别聚合的 `LOW_NUMBER_OF_BYTES_USED` 是每个 `LOW_NUMBER_OF_BYTES_USED` 的总和（假设最坏情况）。同样，较高级别聚合的 `HIGH_NUMBER_OF_BYTES_USED` 是每个 `HIGH_NUMBER_OF_BYTES_USED` 的总和（假设最坏情况）。

11MB 是一个较低的估计值，只有当两个线程同时达到低使用标记时才会发生。

14MB 是一个较高的估计值，只有当两个线程同时达到高使用标记时才会发生。

该账户的实际内存使用量可能在 11.5MB 到 13.5MB 的范围内。

对于容量规划，报告最坏情况实际上是期望的行为，因为它展示了当会话不相关时可能发生的情况，这通常是情况。
