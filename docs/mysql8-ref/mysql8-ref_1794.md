> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-logspaces.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-logspaces.html)

#### 25.6.16.43 `ndbinfo` 日志空间表

该表提供了关于 NDB 集群日志空间使用情况的信息。

`logspaces`表包含以下列：

+   `node_id`

    此数据节点的 ID。

+   `log_type`

    日志类型；可以是`REDO`或`DD-UNDO`之一。

+   `node_id`

    日志 ID；对于磁盘数据撤销日志文件，这与信息模式`FILES`表中的`LOGFILE_GROUP_NUMBER`列显示的值相同，以及`ndbinfo` `logbuffers`表中的`log_id`列显示的值相同

+   `log_part`

    日志部分编号。

+   `total`

    该日志可用的总空间。

+   `used`

    此日志使用的空间。
