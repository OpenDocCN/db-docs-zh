> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-multi-source-start-replica.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-multi-source-start-replica.html)

#### 19.1.5.5 启动多源复制副本

一旦为所有复制源添加了通道，请发出`START REPLICA`（或在 MySQL 8.0.22 之前，`START SLAVE`）语句以开始复制。当您在副本上启用了多个通道时，您可以选择启动所有通道，或选择特定通道启动。例如，要分别启动两个通道，请使用**mysql**客户端发出以下语句：

```sql
mysql> START SLAVE FOR CHANNEL "source_1";
mysql> START SLAVE FOR CHANNEL "source_2";
Or from MySQL 8.0.22:
mysql> START REPLICA FOR CHANNEL "source_1";
mysql> START REPLICA FOR CHANNEL "source_2";
```

要查看`START REPLICA`命令的完整语法和其他可用选项，请参见 Section 15.4.2.6, “START REPLICA Statement”。

要验证两个通道是否已启动并正常运行，您可以在副本上发出`SHOW REPLICA STATUS`语句，例如：

```sql
mysql> SHOW SLAVE STATUS FOR CHANNEL "source_1"\G
mysql> SHOW SLAVE STATUS FOR CHANNEL "source_2"\G
Or from MySQL 8.0.22:
mysql> SHOW REPLICA STATUS FOR CHANNEL "source_1"\G
mysql> SHOW REPLICA STATUS FOR CHANNEL "source_2"\G
```
