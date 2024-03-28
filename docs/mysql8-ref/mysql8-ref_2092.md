> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-clone-progress-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-clone-progress-table.html)

#### 29.12.19.2 `clone_progress` 表

注意

此处描述的性能模式表自 MySQL 8.0.17 起可用。

`clone_progress` 表仅显示当前或最近执行的克隆操作的进度信息。

克隆操作的阶段包括 `DROP DATA`、`FILE COPY`、`PAGE_COPY`、`REDO_COPY`、`FILE_SYNC`、`RESTART` 和 `RECOVERY`。每个阶段都会生成一条记录。因此，表中只包含七行数据，或为空。

`clone_progress` 表具有以下列：

+   `ID`

    当前 MySQL 服务器实例中的唯一克隆操作标识符。

+   `STAGE`

    当前克隆阶段的名称。阶段包括 `DROP DATA`、`FILE COPY`、`PAGE_COPY`、`REDO_COPY`、`FILE_SYNC`、`RESTART` 和 `RECOVERY`。

+   `STATE`

    当前克隆阶段的当前状态。状态包括 `Not Started`、`In Progress` 和 `Completed`。

+   `BEGIN_TIME`

    以`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式的时间戳，显示克隆阶段何时开始。如果阶段尚未开始，则报告 NULL。

+   `END_TIME`

    以`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式的时间戳，显示克隆阶段何时完成。如果阶段尚未结束，则报告 NULL。

+   `THREADS`

    阶段中使用的并发线程数。

+   `ESTIMATE`

    当前阶段的估计数据量，以字节为单位。

+   `DATA`

    当前状态下传输的数据量，以字节为单位。

+   `NETWORK`

    当前状态下传输的网络数据量，以字节为单位。

+   `DATA_SPEED`

    数据传输的当前实际速度，以每秒字节计。此值可能与由`clone_max_data_bandwidth`定义的请求的最大数据传输速率不同。

+   `NETWORK_SPEED`

    当前网络传输速度，以每秒字节计。

`clone_progress` 表是只读的。不允许 DDL，包括`TRUNCATE TABLE`。
