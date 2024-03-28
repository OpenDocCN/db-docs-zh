> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-backup-troubleshooting.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-backup-troubleshooting.html)

#### 25.6.8.4 NDB 集群备份故障排除

如果在发出备份请求时返回错误代码，最有可能的原因是内存或磁盘空间不足。你应该检查备份所需的内存是否足够。

重要提示

如果你已经设置了`BackupDataBufferSize`和`BackupLogBufferSize`，它们的总和大于 4MB，那么你还必须设置`BackupMemory`。

你还应确保备份目标的硬盘分区有足够的空间。

`NDB` 不支持可重复读，这可能会导致恢复过程中出现问题。尽管备份过程是“热”进行的，但从备份中恢复 NDB 集群并非完全“热”过程。这是因为在恢复过程中，正在运行的事务从恢复的数据中获得不可重复读。这意味着在恢复过程中数据的状态是不一致的。
