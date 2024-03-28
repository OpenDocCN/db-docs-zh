> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-disk-write-speed-base.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-disk-write-speed-base.html)

#### 25.6.16.27 ndbinfo disk_write_speed_base 表

`disk_write_speed_base`表提供了有关 LCP、备份和恢复操作期间磁盘写入速度的基本信息。

`disk_write_speed_base`表包含以下列：

+   `node_id`

    此节点的节点 ID

+   `thr_no`

    此 LDM 线程的线程 ID

+   `millis_ago`

    自此报告期结束以来的毫秒数

+   `millis_passed`

    在此报告期间经过的毫秒数

+   `backup_lcp_bytes_written`

    在此期间由本地检查点和备份过程写入磁盘的字节数

+   `redo_bytes_written`

    在此期间写入 REDO 日志的字节数

+   `target_disk_write_speed`

    每个 LDM 线程的实际磁盘写入速度（基础数据）
