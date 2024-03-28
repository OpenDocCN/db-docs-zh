> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-performance-timers-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-performance-timers-table.html)

#### 29.12.21.6 The performance_timers Table

`performance_timers` 表显示了可用的事件计时器：

```sql
mysql> SELECT * FROM performance_schema.performance_timers;
+-------------+-----------------+------------------+----------------+
| TIMER_NAME  | TIMER_FREQUENCY | TIMER_RESOLUTION | TIMER_OVERHEAD |
+-------------+-----------------+------------------+----------------+
| CYCLE       |      2389029850 |                1 |             72 |
| NANOSECOND  |      1000000000 |                1 |            112 |
| MICROSECOND |         1000000 |                1 |            136 |
| MILLISECOND |            1036 |                1 |            168 |
| THREAD_CPU  |       339101694 |                1 |            798 |
+-------------+-----------------+------------------+----------------+
```

如果与特定计时器名称相关联的值为 `NULL`，则该计时器在您的平台上不受支持。有关事件计时发生的解释，请参见 Section 29.4.1, “Performance Schema Event Timing”。

`performance_timers` 表具有以下列：

+   `TIMER_NAME`

    计时器名称。

+   `TIMER_FREQUENCY`

    每秒的计时器单位数。对于循环计时器，频率通常与 CPU 速度有关。例如，在一个具有 2.4GHz 处理器的系统上，`CYCLE` 可能接近 2400000000。

+   `TIMER_RESOLUTION`

    指示计时器值增加的计时器单位数。如果计时器的分辨率为 10，则每次增加 10。

+   `TIMER_OVERHEAD`

    通过调用计时器 20 次进行初始化并选择最小值，性能模式确定获得给定计时器的一个计时所需的最小周期数。总开销实际上是这个值的两倍，因为在每个事件的开始和结束时，仪器会调用计时器。计时器代码仅在计时事件时调用，因此此开销不适用于非计时事件。

`performance_timers` 表没有索引。

`TRUNCATE TABLE` 不允许用于 `performance_timers` 表。
