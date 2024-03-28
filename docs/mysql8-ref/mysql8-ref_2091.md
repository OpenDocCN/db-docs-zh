> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-clone-status-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-clone-status-table.html)

#### 29.12.19.1 克隆状态表

注意

此处描述的性能模式表自 MySQL 8.0.17 起可用。

`clone_status`表仅显示当前或最后执行的克隆操作的状态。该表只包含一行数据，或为空。

`clone_status`表包含以下列：

+   `ID`

    当前 MySQL 服务器实例中的唯一克隆操作标识符。

+   `PID`

    执行克隆操作的会话的进程列表 ID。

+   `STATE`

    克隆操作的当前状态。值包括`Not Started`、`In Progress`、`Completed`和`Failed`。

+   `BEGIN_TIME`

    克隆操作开始时的时间戳格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'。

+   `END_TIME`

    克隆操作完成时的时间戳格式为`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'。如果操作尚未结束，则报告 NULL。

+   `SOURCE`

    源 MySQL 服务器地址以'`HOST:PORT`'格式显示。对于本地克隆操作，该列显示'`LOCAL INSTANCE`'。

+   `DESTINATION`

    正在进行克隆的目录。

+   `ERROR_NO`

    克隆操作失败时报告的错误编号。

+   `ERROR_MESSAGE`

    克隆操作失败时的错误消息字符串。

+   `BINLOG_FILE`

    数据被克隆到的二进制日志文件的名称。

+   `BINLOG_POSITION`

    数据被克隆到的二进制日志文件偏移量。

+   `GTID_EXECUTED`

    最后一个克隆事务的 GTID 值。

`clone_status`表是只读的。不允许 DDL，包括`TRUNCATE TABLE`。
