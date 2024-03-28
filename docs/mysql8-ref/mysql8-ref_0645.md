# 10.14.7 复制连接线程状态

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replica-connection-thread-states.html`](https://dev.mysql.com/doc/refman/8.0/en/replica-connection-thread-states.html)

这些线程状态发生在副本服务器上，但与连接线程相关，而不是与 I/O 或 SQL 线程相关。

在 MySQL 8.0.26 中，对仪表名称进行了不兼容的更改，包括线程阶段的名称，包含术语“master”，更改为“source”，“slave”，更改为“replica”，以及“mts”（用于“多线程从属”），更改为“mta”（用于“多线程应用程序”）。使用这些仪表名称的监控工具可能会受到影响。如果不兼容的更改对您产生影响，请将`terminology_use_previous`系统变量设置为`BEFORE_8_0_26`，以使 MySQL Server 使用上述列表中指定对象的旧版本名称。这样可以使依赖旧名称的监控工具继续工作，直到它们可以更新为使用新名称。

将`terminology_use_previous`系统变量设置为会话范围以支持个别功能，或者设置为全局范围以成为所有新会话的默认值。当使用全局范围时，慢查询日志包含旧版本的名称。

+   `Changing master`

    从 MySQL 8.0.26 开始：`Changing replication source`

    线程正在处理`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）。

+   `Killing slave`

    线程正在处理`STOP REPLICA`语句。

+   `Opening master dump table`

    此状态发生在`Creating table from master dump`之后。

+   `Reading master dump table data`

    此状态发生在`Opening master dump table`之后。

+   `Rebuilding the index on master dump table`

    此状态发生在`Reading master dump table data`之后。
