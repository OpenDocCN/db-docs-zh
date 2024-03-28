> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-disk-write-speed-aggregate-node.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-disk-write-speed-aggregate-node.html)

#### 25.6.16.29 ndbinfo disk_write_speed_aggregate_node 表

`disk_write_speed_aggregate_node`表提供了有关在 LCP、备份和恢复操作期间磁盘写入速度的每个节点的汇总信息。

`disk_write_speed_aggregate_node`表包含以下列：

+   `node_id`

    此节点的节点 ID

+   `backup_lcp_speed_last_sec`

    上一秒由备份和 LCP 进程写入磁盘的字节数

+   `redo_speed_last_sec`

    上一秒写入重做日志的字节数

+   `backup_lcp_speed_last_10sec`

    每秒由备份和 LCP 进程写入磁盘的字节数，平均值为过去 10 秒

+   `redo_speed_last_10sec`

    每秒写入重做日志的字节数，平均值为过去 10 秒

+   `backup_lcp_speed_last_60sec`

    每秒由备份和 LCP 进程写入磁盘的字节数，平均值为过去 60 秒

+   `redo_speed_last_60sec`

    每秒写入重做日志的字节数，平均值为过去 60 秒
