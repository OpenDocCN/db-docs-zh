> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-disk-write-speed-aggregate.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-disk-write-speed-aggregate.html)

#### 25.6.16.28 ndbinfo disk_write_speed_aggregate 表

`disk_write_speed_aggregate`表提供了有关 LCP、备份和恢复操作期间磁盘写入速度的聚合信息。

`disk_write_speed_aggregate`表包含以下列：

+   `node_id`

    此节点的节点 ID

+   `thr_no`

    此 LDM 线程的线程 ID

+   `backup_lcp_speed_last_sec`

    备份和 LCP 进程上一秒写入磁盘的字节数

+   `redo_speed_last_sec`

    上一秒写入 REDO 日志的字节数

+   `backup_lcp_speed_last_10sec`

    备份和 LCP 进程每秒写入磁盘的字节数，过去 10 秒的平均值

+   `redo_speed_last_10sec`

    每秒写入 REDO 日志的字节数，过去 10 秒的平均值

+   `std_dev_backup_lcp_speed_last_10sec`

    备份和 LCP 进程每秒写入磁盘的字节数的标准偏差，过去 10 秒的平均值

+   `std_dev_redo_speed_last_10sec`

    每秒写入 REDO 日志的字节数的标准偏差，过去 10 秒的平均值

+   `backup_lcp_speed_last_60sec`

    备份和 LCP 进程每秒写入磁盘的字节数，过去 60 秒的平均值

+   `redo_speed_last_60sec`

    每秒写入 REDO 日志的字节数，过去 10 秒的平均值

+   `std_dev_backup_lcp_speed_last_60sec`

    备份和 LCP 进程每秒写入磁盘的字节数的标准偏差，过去 60 秒的平均值

+   `std_dev_redo_speed_last_60sec`

    每秒写入 REDO 日志的字节数的标准偏差，过去 60 秒的平均值

+   `slowdowns_due_to_io_lag`

    自上次节点启动以来，由于 REDO 日志 I/O 延迟而导致磁盘写入减速的秒数

+   `slowdowns_due_to_high_cpu`

    自上次节点启动以来，由于高 CPU 使用率而导致磁盘写入减速的秒数

+   `disk_write_speed_set_to_min`

    自上次节点启动以来，磁盘写入速度被设置为最小的秒数

+   `current_target_disk_write_speed`

    每个 LDM 线程的实际磁盘写入速度（聚合）
