> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-processlist.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-processlist.html)

#### 30.4.3.22 The processlist and x$processlist Views

MySQL 进程列表显示服务器内执行的线程集合当前正在执行的操作。`processlist` 和 `x$processlist` 视图总结了进程信息。它们提供的信息比 `SHOW PROCESSLIST` 语句和 `INFORMATION_SCHEMA` `PROCESSLIST` 表更完整，而且不会阻塞。默认情况下，行按照进程时间和等待时间降序排序。有关进程信息来源的比较，请参见 进程信息来源。

这里的列描述很简要。有关更多信息，请参阅性能模式 `threads` 表的描述，链接在 第 29.12.21.8 节，“The threads Table”。

`processlist` 和 `x$processlist` 视图具有以下列：

+   `thd_id`

    线程 ID。

+   `conn_id`

    连接 ID。

+   `user`

    线程用户或线程名称。

+   `db`

    线程的默认数据库，如果没有则为`NULL`。

+   `command`

    对于前台线程，线程代表客户端执行的命令类型，如果会话空闲，则为`Sleep`。

+   `state`

    指示线程正在执行的操作、事件或状态。

+   `time`

    线程在当前状态下已经经过的秒数。

+   `current_statement`

    线程正在执行的语句，如果没有执行任何语句则为`NULL`。

+   `execution_engine`

    查询执行引擎。该值为`PRIMARY`或`SECONDARY`。用于 MySQL HeatWave 服务和 HeatWave，其中`PRIMARY`引擎为`InnoDB`，`SECONDARY`引擎为 HeatWave（`RAPID`）。对于 MySQL Community Edition Server、MySQL Enterprise Edition Server（本地）和没有 HeatWave 的 MySQL HeatWave 服务，该值始终为`PRIMARY`。此列在 MySQL 8.0.29 中添加。

+   `statement_latency`

    语句执行的时间长。

+   `progress`

    支持进度报告的阶段完成的工作百分比。参见 第 30.3 节，“sys Schema Progress Reporting”。

+   `lock_latency`

    当前语句等待锁的时间。

+   `cpu_latency`

    当前线程在 CPU 上花费的时间。

+   `rows_examined`

    当前语句从存储引擎中读取的行数。

+   `rows_sent`

    当前语句返回的行数。

+   `rows_affected`

    当前语句影响的行数。

+   `tmp_tables`

    当前语句创建的内存中临时表的数量。

+   `tmp_disk_tables`

    当前语句创建的磁盘上临时表的数量。

+   `full_scan`

    当前语句执行的全表扫描次数。

+   `last_statement`

    线程执行的最后一条语句，如果当前没有正在执行的语句或等待。

+   `last_statement_latency`

    最后一条语句执行的时间。

+   `current_memory`

    线程分配的字节数。

+   `last_wait`

    线程的最近等待事件名称。

+   `last_wait_latency`

    线程的最近等待事件的等待时间。

+   `source`

    包含产生事件的受检代码的源文件和行号。

+   `trx_latency`

    线程的当前事务等待时间。

+   `trx_state`

    线程的当前事务状态。

+   `trx_autocommit`

    当前事务开始时是否启用了自动提交模式。

+   `pid`

    客户端进程 ID。

+   `program_name`

    客户端程序名称。
