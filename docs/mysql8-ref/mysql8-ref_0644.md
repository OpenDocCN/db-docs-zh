# 10.14.6 复制 SQL 线程状态

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replica-sql-thread-states.html`](https://dev.mysql.com/doc/refman/8.0/en/replica-sql-thread-states.html)

以下列表显示了在复制品服务器上的复制 SQL 线程的`State`列中可能看到的最常见状态。

在 MySQL 8.0.26 中，对仪表化名称进行了不兼容的更改，包括包含术语“master”的线程阶段名称，更改为“source”，“slave”更改为“replica”，以及“mts”（用于“多线程复制品”）更改为“mta”（用于“多线程应用程序”）。使用这些仪表化名称的监控工具可能会受到影响。如果不兼容的更改对您产生影响，请将`terminology_use_previous`系统变量设置为`BEFORE_8_0_26`，以使 MySQL Server 使用前面列表中指定对象的旧名称。这使依赖旧名称的监控工具可以继续工作，直到它们可以更���为使用新名称。

将`terminology_use_previous`系统变量设置为会话范围以支持个别功能，或者设置为全局范围以成为所有新会话的默认值。当使用全局范围时，慢查询日志包含旧版本的名称。

+   `Making temporary file (append) before replaying LOAD DATA INFILE`

    线程正在执行一个`LOAD DATA`语句，并将数据附加到一个临时文件中，该文件包含复制品读取行的数据。

+   `Making temporary file (create) before replaying LOAD DATA INFILE`

    线程正在执行一个`LOAD DATA`语句，并创建一个包含复制品读取行的数据的临时文件。只有在原始`LOAD DATA`语句由运行版本低于 MySQL 5.0.3 的源记录时，才会遇到此状态。

+   `Reading event from the relay log`

    线程已从中继日志中读取事件，以便可以处理该事件。

+   `Slave has read all relay log; waiting for more updates`

    从 MySQL 8.0.26 开始：`Replica has read all relay log; waiting for more updates`

    线程已处理完中继日志文件中的所有事件，现在正在等待 I/O（接收器）线程将新事件写入中继日志。

+   `Waiting for an event from Coordinator`

    使用多线程复制品（`replica_parallel_workers`或`slave_parallel_workers`大于 1），其中一个复制品工作线程正在等待来自协调器线程的事件。

+   `Waiting for slave mutex on exit`

    从 MySQL 8.0.26 开始：`Waiting for replica mutex on exit`

    作为线程停止时发生的非常简短的状态。

+   `等待从属工作者释放待处理事件`

    从 MySQL 8.0.26 开始：`等待复制工作者释放待处理事件`

    当工作者处理的事件总大小超过`replica_pending_jobs_size_max`或`slave_pending_jobs_size_max`系统变量的大小时，会发生这种等待动作。当大小低于此限制时，协调器会恢复调度。只有当`replica_parallel_workers`或`slave_parallel_workers`设置大于 0 时才会发生这种状态。

+   `等待在中继日志中的下一个事件`

    在`从中继日志中读取事件`之前的初始状态。

+   `等待直到主服务器执行事件后的 MASTER_DELAY 秒`

    从 MySQL 8.0.26 开始：`等待直到主服务器执行事件后的 SOURCE_DELAY 秒`

    SQL 线程已经读取了一个事件，但正在等待复制延迟结束。这个延迟是通过`CHANGE REPLICATION SOURCE TO`语句（MySQL 8.0.23 之后）或`CHANGE MASTER TO`语句（MySQL 8.0.23 之前）中的`SOURCE_DELAY` | `MASTER_DELAY`选项设置的。

SQL 线程的`Info`列也可能显示一个语句的文本。这表示线程已经从中继日志中读取了一个事件，提取了其中的语句，并可能正在执行它。
