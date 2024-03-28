# 9.3.3 备份策略摘要

> 原文：[`dev.mysql.com/doc/refman/8.0/en/backup-strategy-summary.html`](https://dev.mysql.com/doc/refman/8.0/en/backup-strategy-summary.html)

在操作系统崩溃或断电的情况下，`InnoDB`本身会完成所有数据恢复的工作。但为了确保您能安心入睡，请遵守以下准则：

+   始终以启用二进制日志记录的方式运行 MySQL 服务器（这是 MySQL 8.0 的默认设置）。如果您有这样的安全介质，这种技术也可以用于磁盘负载平衡（从而提高性能）。

+   做定期完整备份，使用之前在第 9.3.1 节，“建立备份策略”中展示的**mysqldump**命令，进行在线、非阻塞备份。

+   通过使用`FLUSH LOGS`或**mysqladmin flush-logs**来进行定期增量备份。
