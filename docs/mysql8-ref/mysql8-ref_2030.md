> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-events-stages-current-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-stages-current-table.html)

#### 29.12.5.1 events_stages_current 表

`events_stages_current`表包含当前阶段事件。该表为每个线程存储一行，显示线程最近监视的阶段事件的当前状态，因此没有用于配置表大小的系统变量。

包含阶段事件行的表中，`events_stages_current`是最基本的。其他包含阶段事件行的表是从当前事件逻辑推导出来的。例如，`events_stages_history`和`events_stages_history_long`表是最近已结束的阶段事件的集合，每个线程和全局跨所有线程的最大行数。

有关三个阶段事件表之间关系的更多信息，请参见第 29.9 节“当前和历史事件的性能模式表”。

有关配置是否收集阶段事件的信息，请参见第 29.12.5 节“性能模式阶段事件表”。

`events_stages_current`表具有以下列：

+   `THREAD_ID`，`EVENT_ID`

    与事件相关联的线程以及事件开始时的线程当前事件编号。`THREAD_ID`和`EVENT_ID`值一起唯一标识行。没有两行具有相同的值对。

+   `END_EVENT_ID`

    当事件开始时，此列设置为`NULL`，并在事件结束时更新为线程当前事件编号。

+   `EVENT_NAME`

    产生事件的仪器名称。这是`setup_instruments`表中的一个`NAME`值。仪器名称可能由多个部分组成，形成一个层次结构，如第 29.6 节“性能模式仪器命名约定”所讨论的那样。

+   `SOURCE`

    包含产生事件的被检测代码的源文件名和文件中发生检测的行号。这使您可以检查源代码以确定涉及的确切代码。

+   `TIMER_START`、`TIMER_END`、`TIMER_WAIT`

    事件的时间信息。这些值的单位为皮秒（万亿分之一秒）。`TIMER_START`和`TIMER_END`值表示事件计时开始和结束的时间。`TIMER_WAIT`是事件经过的时间（持续时间）。

    如果事件尚未完成，`TIMER_END`是当前计时器值，`TIMER_WAIT`是到目前为止经过的时间（`TIMER_END` − `TIMER_START`）。

    如果事件是由`TIMED = NO`的工具产生的，则不会收集时间信息，`TIMER_START`、`TIMER_END`和`TIMER_WAIT`都为`NULL`。

    有关皮秒作为事件时间单位以及影响时间值的因素的讨论，请参阅第 29.4.1 节，“性能模式事件定时”。

+   `WORK_COMPLETED`、`WORK_ESTIMATED`

    这些列提供阶段进度信息，用于生成此类信息的工具。`WORK_COMPLETED`指示已完成的阶段工作单元数，`WORK_ESTIMATED`指示阶段预期的工作单元数。有关更多信息，请参阅阶段事件进度信息。

+   `NESTING_EVENT_ID`

    此事件嵌套在其中的事件的`EVENT_ID`值。阶段事件的嵌套事件通常是语句事件。

+   `NESTING_EVENT_TYPE`

    嵌套事件类型。其值为`TRANSACTION`、`STATEMENT`、`STAGE`或`WAIT`。

`events_stages_current`表具有以下索引：

+   主键为(`THREAD_ID`, `EVENT_ID`)

`TRUNCATE TABLE`允许用于`events_stages_current`表。它会删除行。
