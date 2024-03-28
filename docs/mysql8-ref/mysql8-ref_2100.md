> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-file-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-file-summary-tables.html)

#### 29.12.20.7 文件 I/O 汇总表

性能模式维护文件 I/O 汇总表，汇总有关 I/O 操作的信息。

示例文件 I/O 事件汇总信息：

```sql
mysql> SELECT * FROM performance_schema.file_summary_by_event_name\G
...
*************************** 2\. row ***************************
               EVENT_NAME: wait/io/file/sql/binlog
               COUNT_STAR: 31
           SUM_TIMER_WAIT: 8243784888
           MIN_TIMER_WAIT: 0
           AVG_TIMER_WAIT: 265928484
           MAX_TIMER_WAIT: 6490658832
...
mysql> SELECT * FROM performance_schema.file_summary_by_instance\G
...
*************************** 2\. row ***************************
                FILE_NAME: /var/mysql/share/english/errmsg.sys
               EVENT_NAME: wait/io/file/sql/ERRMSG
               EVENT_NAME: wait/io/file/sql/ERRMSG
    OBJECT_INSTANCE_BEGIN: 4686193384
               COUNT_STAR: 5
           SUM_TIMER_WAIT: 13990154448
           MIN_TIMER_WAIT: 26349624
           AVG_TIMER_WAIT: 2798030607
           MAX_TIMER_WAIT: 8150662536
...
```

每个文件 I/O 汇总表都有一个或多个分组列，指示表如何聚合事件。事件名称指的是`setup_instruments`表中事件仪器的名称：

+   `file_summary_by_event_name`有一个`EVENT_NAME`列。每行总结了给定事件名称的事件。

+   `file_summary_by_instance`有`FILE_NAME`、`EVENT_NAME`和`OBJECT_INSTANCE_BEGIN`列。每行总结了给定文件和事件名称的事件。

每个文件 I/O 汇总表都有以下包含聚合值的汇总列。一些列更通用，其值与更细粒度列的值之和相同。通过这种方式，高级别的聚合可以直接使用，无需用户定义的视图来汇总较低级别的列。

+   `COUNT_STAR`、`SUM_TIMER_WAIT`、`MIN_TIMER_WAIT`、`AVG_TIMER_WAIT`、`MAX_TIMER_WAIT`

    这些列汇总所有 I/O 操作。

+   `COUNT_READ`、`SUM_TIMER_READ`、`MIN_TIMER_READ`、`AVG_TIMER_READ`、`MAX_TIMER_READ`、`SUM_NUMBER_OF_BYTES_READ`

    这些列汇总所有读操作，包括`FGETS`、`FGETC`、`FREAD`和`READ`。

+   `COUNT_WRITE`、`SUM_TIMER_WRITE`、`MIN_TIMER_WRITE`、`AVG_TIMER_WRITE`、`MAX_TIMER_WRITE`、`SUM_NUMBER_OF_BYTES_WRITE`

    这些列汇总所有写操作，包括`FPUTS`、`FPUTC`、`FPRINTF`、`VFPRINTF`、`FWRITE`和`PWRITE`。

+   `COUNT_MISC`、`SUM_TIMER_MISC`、`MIN_TIMER_MISC`、`AVG_TIMER_MISC`、`MAX_TIMER_MISC`

    这些列汇总所有其他 I/O 操作，包括`CREATE`、`DELETE`、`OPEN`、`CLOSE`、`STREAM_OPEN`、`STREAM_CLOSE`、`SEEK`、`TELL`、`FLUSH`、`STAT`、`FSTAT`、`CHSIZE`、`RENAME`和`SYNC`。这些操作没有字节计数。

文件 I/O 汇总表具有以下索引：

+   `file_summary_by_event_name`:

    +   主键在(`EVENT_NAME`)

+   `file_summary_by_instance`:

    +   主键在(`OBJECT_INSTANCE_BEGIN`)

    +   索引在(`FILE_NAME`)

    +   索引在(`EVENT_NAME`)

`TRUNCATE TABLE`允许用于文件 I/O 汇总表。它将汇总列重置为零，而不是删除行。

MySQL 服务器使用多种技术来避免通过缓存从文件中读取的信息进行 I/O 操作，因此可能会出现您预期会导致 I/O 事件的语句实际上并未执行 I/O 操作。您可以通过刷新缓存或重新启动服务器来确保发生 I/O 操作以重置其状态。
