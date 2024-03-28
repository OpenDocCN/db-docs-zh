> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-component-scheduler-tasks-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-component-scheduler-tasks-table.html)

#### 29.12.21.1 组件调度器任务表

`component_scheduler_tasks`表为每个预定任务包含一行。每行包含关于应用程序、组件和插件可以实现的任务的进行中进展的信息，可选地使用`scheduler`组件（参见第 7.5.5 节，“调度器组件”）。例如，`audit_log`服务器插件利用`scheduler`组件定期运行其内存缓存的刷新：

```sql
mysql> select * from performance_schema.component_scheduler_tasks\G
*************************** 1\. row ***************************
            NAME: plugin_audit_log_flush_scheduler
          STATUS: WAITING
         COMMENT: Registered by the audit log plugin. Does a periodic refresh of the audit log 
                  in-memory rules cache by calling audit_log_flush
INTERVAL_SECONDS: 100
       TIMES_RUN: 5
    TIMES_FAILED: 0 1 row in set (0.02 sec)
```

`component_scheduler_tasks`表具有以下列：

+   `NAME`

    在注册时提供的名称。

+   `STATUS`

    值为：

    +   如果任务处于活动状态并正在执行，则为`RUNNING`。

    +   如果任务处于空闲状态并等待后台线程接手或等待下一次运行时间到来，则为`WAITING`。

+   `COMMENT`

    由应用程序、组件或插件提供的编译时注释。在前面的示例中，MySQL 企业审计使用名为`audit_log`的服务器插件提供注释。

+   `INTERVAL_SECONDS`

    以秒为单位运行任务的时间，由应用程序、组件或插件提供。MySQL 企业审计允许您使用`audit_log_flush_interval_seconds`系统变量指定此值。

+   `TIMES_RUN`

    每次任务成功运行时递增的计数器。它会循环。

+   `TIMES_FAILED`

    每次任务执行失败时递增的计数器。它会循环。
