> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-logbuffers.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-logbuffers.html)

#### 25.6.16.42 `ndbinfo` 日志缓冲区表

`logbuffer` 表提供了关于 NDB 集群日志缓冲区使用情况的信息。

`logbuffers` 表包含以下列：

+   `node_id`

    此数据节点的 ID。

+   `log_type`

    日志类型。其中之一：`REDO`、`DD-UNDO`、`BACKUP-DATA` 或 `BACKUP-LOG`。

+   `log_id`

    日志 ID；对于磁盘数据撤销日志文件，这与信息模式 `FILES` 表中的 `LOGFILE_GROUP_NUMBER` 列显示的值以及 `ndbinfo` `logspaces` 表的 `log_id` 列显示的值相同

+   `log_part`

    日志部分编号

+   `total`

    此日志可用的总空间

+   `used`

    此日志使用的空间

##### 笔记

在执行 NDB 备份时，`logbuffers` 表行反映了另外两种日志类型。其中一行具有日志类型 `BACKUP-DATA`，显示了在备份期间用于将片段复制到备份文件的数据缓冲区的使用量。另一行具有日志类型 `BACKUP-LOG`，显示了在备份开始后记录所做更改时备份期间使用的日志缓冲区的使用量。在集群中的每个数据节点的 `logbuffers` 表中分别显示了这些 `log_type` 行。除非当前正在执行 NDB 备份，否则这些行不会出现。
