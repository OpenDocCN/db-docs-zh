# 19.4.3 监视基于行的复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-solutions-rbr-monitoring.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-solutions-rbr-monitoring.html)

当使用基于行的复制时，通过性能模式仪表阶段监视复制应用程序（SQL）线程的当前进度，使您能够跟踪操作的处理并检查已完成的工作量和估计的工作量。当启用这些性能模式仪表阶段时，`events_stages_current` 表显示了应用程序线程的阶段及其进度。有关背景信息，请参阅第 29.12.5 节，“性能模式阶段事件表”。

要跟踪所有三种基于行的复制事件类型（写入、更新、删除）的进度：

+   通过执行以下命令启用三个性能模式阶段：

    ```sql
    mysql> UPDATE performance_schema.setup_instruments SET ENABLED = 'YES'
     -> WHERE NAME LIKE 'stage/sql/Applying batch of row changes%';
    ```

+   等待一些事件被复制应用程序线程处理，然后通过查看`events_stages_current`表来检查进度。例如，要获取`update`事件的进度，请执行：

    ```sql
    mysql> SELECT WORK_COMPLETED, WORK_ESTIMATED FROM performance_schema.events_stages_current
     -> WHERE EVENT_NAME LIKE 'stage/sql/Applying batch of row changes (update)'
    ```

+   如果启用了`binlog_rows_query_log_events`，则会将有关查询的信息存储在二进制日志中，并在`processlist_info`字段中公开。要查看触发此事件的原始查询：

    ```sql
    mysql> SELECT db, processlist_state, processlist_info FROM performance_schema.threads
     -> WHERE processlist_state LIKE 'stage/sql/Applying batch of row changes%' AND thread_id = N;
    ```
