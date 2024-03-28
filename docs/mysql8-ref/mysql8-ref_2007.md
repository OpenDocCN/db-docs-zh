# 29.8 性能模式原子和分子事件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-atom-molecule-events.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-atom-molecule-events.html)

对于表 I/O 事件，在`events_waits_current`中通常有两行，而不是一行。例如，行提取可能导致类似以下的行：

```sql
Row# EVENT_NAME                 TIMER_START TIMER_END
---- ----------                 ----------- ---------
   1 wait/io/file/myisam/dfile        10001 10002
   2 wait/io/table/sql/handler        10000 NULL
```

行提取导致文件读取。在这个例子中，表 I/O 提取事件在文件 I/O 事件之前开始，但尚未完成（其`TIMER_END`值为`NULL`）。文件 I/O 事件“嵌套”在表 I/O 事件中。

这是因为，与其他“原子”等待事件（如互斥或文件 I/O）不同，表 I/O 事件是“分子”事件，并包括（重叠）其他事件。在`events_waits_current`中，表 I/O 事件通常有两行：

+   最近的表 I/O 等待事件的一行

+   任何类型的最新等待事件的一行

通常情况下，“任何类型”的等待事件与表 I/O 事件不同。随着每个子事件的完成，它会从`events_waits_current`中消失。在此时，直到下一个子事件开始之前，表 I/O 等待也是任何类型的最新等待。
