> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-multi-source-adding-binlog-master.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-multi-source-adding-binlog-master.html)

#### 19.1.5.4 将基于二进制日志的复制源添加到多源复制副本

这些步骤假定源上已启用二进制日志记录（这是默认设置），副本正在使用基于`TABLE`的复制应用程序元数据存储库（这是 MySQL 8.0 中的默认设置），并且您已启用了一个复制用户并记录了当前的二进制日志文件名和位置。

使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）为副本上的每个源配置一个复制通道（请参见 Section 19.2.2, “Replication Channels”）。`FOR CHANNEL`子句用于指定通道。例如，要将`source1`和`source2`添加为副本的源，请使用**mysql**客户端在副本上两次发出该语句，如下所示：

```sql
mysql> CHANGE MASTER TO MASTER_HOST="source1", MASTER_USER="ted", MASTER_PASSWORD="*password*", \
MASTER_LOG_FILE='source1-bin.000006', MASTER_LOG_POS=628 FOR CHANNEL "source_1";
mysql> CHANGE MASTER TO MASTER_HOST="source2", MASTER_USER="ted", MASTER_PASSWORD="*password*", \
MASTER_LOG_FILE='source2-bin.000018', MASTER_LOG_POS=104 FOR CHANNEL "source_2";

Or from MySQL 8.0.23:
mysql> CHANGE REPLICATION SOURCE TO SOURCE_HOST="source1", SOURCE_USER="ted", SOURCE_PASSWORD="*password*", \
SOURCE_LOG_FILE='source1-bin.000006', SOURCE_LOG_POS=628 FOR CHANNEL "source_1";
mysql> CHANGE REPLICATION SOURCE TO SOURCE_HOST="source2", SOURCE_USER="ted", SOURCE_PASSWORD="*password*", \
SOURCE_LOG_FILE='source2-bin.000018', SOURCE_LOG_POS=104 FOR CHANNEL "source_2";
```

要使副本仅从`source1`复制数据库`db1`，并且仅从`source2`复制数据库`db2`，请使用**mysql**客户端为每个通道发出`CHANGE REPLICATION FILTER`语句，如下所示：

```sql
mysql> CHANGE REPLICATION FILTER REPLICATE_WILD_DO_TABLE = ('db1.%') FOR CHANNEL "source_1";
mysql> CHANGE REPLICATION FILTER REPLICATE_WILD_DO_TABLE = ('db2.%') FOR CHANNEL "source_2";
```

对于`CHANGE REPLICATION FILTER`语句的完整语法和其他可用选项，请参见 Section 15.4.2.2, “CHANGE REPLICATION FILTER Statement”。
