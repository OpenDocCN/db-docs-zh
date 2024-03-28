> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-backup-id.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-backup-id.html)

#### 25.6.16.3 `ndbinfo backup_id` 表

该表提供了一种查找最近为该集群启动的备份的 ID 的方法。

`backup_id` 表包含一个名为 `id` 的单列，该列对应使用 **ndb_mgm** 客户端执行 `START BACKUP` 命令进行的备份 ID。该表包含一行。

*示例*: 假设在 NDB 管理客户端中发出以下一系列 `START BACKUP` 命令，自集群首次启动以来没有进行其他备份：

```sql
ndb_mgm> START BACKUP
Waiting for completed, this may take several minutes
Node 5: Backup 1 started from node 50
Node 5: Backup 1 started from node 50 completed
 StartGCP: 27894 StopGCP: 27897
 #Records: 2057 #LogRecords: 0
 Data: 51580 bytes Log: 0 bytes
ndb_mgm> START BACKUP 5
Waiting for completed, this may take several minutes
Node 5: Backup 5 started from node 50
Node 5: Backup 5 started from node 50 completed
 StartGCP: 27905 StopGCP: 27908
 #Records: 2057 #LogRecords: 0
 Data: 51580 bytes Log: 0 bytes
ndb_mgm> START BACKUP
Waiting for completed, this may take several minutes
Node 5: Backup 6 started from node 50
Node 5: Backup 6 started from node 50 completed
 StartGCP: 27912 StopGCP: 27915
 #Records: 2057 #LogRecords: 0
 Data: 51580 bytes Log: 0 bytes
ndb_mgm> START BACKUP 3
Connected to Management Server at: localhost:1186
Waiting for completed, this may take several minutes
Node 5: Backup 3 started from node 50
Node 5: Backup 3 started from node 50 completed
 StartGCP: 28149 StopGCP: 28152
 #Records: 2057 #LogRecords: 0
 Data: 51580 bytes Log: 0 bytes
ndb_mgm>
```

之后，使用 **mysql** 客户端，`backup_id` 表包含如下所示的单行：

```sql
mysql> USE ndbinfo;

Database changed
mysql> SELECT * FROM backup_id;
+------+
| id   |
+------+
|    3 |
+------+
1 row in set (0.00 sec)
```

如果找不到任何备份，则表中包含一个 `id` 值为 `0` 的单行。

`backup_id` 表在 NDB 8.0.24 中添加。
