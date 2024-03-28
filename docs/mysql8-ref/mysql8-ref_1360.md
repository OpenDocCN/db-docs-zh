> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-multi-source-adding-gtid-master.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-multi-source-adding-gtid-master.html)

#### 19.1.5.3 向多源副本添加基于 GTID 的源

这些步骤假定您已经在源上使用`gtid_mode=ON`启用了事务的 GTIDs，创建了一个复制用户，确保副本正在使用基于`TABLE`的复制应用程序元数据存储库，并根据需要为副本提供了来自源的数据。

使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（MySQL 8.0.23 之前）在副本上为每个源配置一个复制通道（参见 Section 19.2.2, “Replication Channels”）。`FOR CHANNEL`子句用于指定通道。对于基于 GTID 的复制，使用 GTID 自动定位来与源进行同步（参见 Section 19.1.3.3, “GTID Auto-Positioning”）。设置`SOURCE_AUTO_POSITION` | `MASTER_AUTO_POSITION`选项以指定使用自动定位。

例如，要将`source1`和`source2`添加为副本的源，请使用**mysql**客户端在副本上发出两次语句，如下所示：

```sql
mysql> CHANGE MASTER TO MASTER_HOST="source1", MASTER_USER="ted", \
MASTER_PASSWORD="*password*", MASTER_AUTO_POSITION=1 FOR CHANNEL "source_1";
mysql> CHANGE MASTER TO MASTER_HOST="source2", MASTER_USER="ted", \
MASTER_PASSWORD="*password*", MASTER_AUTO_POSITION=1 FOR CHANNEL "source_2";

Or from MySQL 8.0.23:
mysql> CHANGE REPLICATION SOURCE TO SOURCE_HOST="source1", SOURCE_USER="ted", \
SOURCE_PASSWORD="*password*", SOURCE_AUTO_POSITION=1 FOR CHANNEL "source_1";
mysql> CHANGE REPLICATION SOURCE TO SOURCE_HOST="source2", SOURCE_USER="ted", \
SOURCE_PASSWORD="*password*", SOURCE_AUTO_POSITION=1 FOR CHANNEL "source_2";
```

要使副本仅从`source1`复制数据库`db1`，并且仅从`source2`复制数据库`db2`，请使用**mysql**客户端为每个通道发出`CHANGE REPLICATION FILTER`语句，如下所示：

```sql
mysql> CHANGE REPLICATION FILTER REPLICATE_WILD_DO_TABLE = ('db1.%') FOR CHANNEL "source_1";
mysql> CHANGE REPLICATION FILTER REPLICATE_WILD_DO_TABLE = ('db2.%') FOR CHANNEL "source_2";
```

有关`CHANGE REPLICATION FILTER`语句的完整语法和其他可用选项，请参见 Section 15.4.2.2, “CHANGE REPLICATION FILTER Statement”。
