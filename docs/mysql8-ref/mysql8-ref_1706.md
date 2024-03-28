> 原文：[`dev.mysql.com/doc/refman/8.0/en/ndb-restore-parallel-data-node-backup.html`](https://dev.mysql.com/doc/refman/8.0/en/ndb-restore-parallel-data-node-backup.html)

#### 25.5.23.3 从并行进行的备份中恢复

NDB Cluster 8.0 支持在每个数据节点上使用**ndbmtd**进行并行备份，具有多个 LDMs（参见第 25.6.8.5 节，“使用并行数据节点进行 NDB 备份”）。接下来的两节描述了如何恢复以这种方式进行的备份。

##### 25.5.23.3.1 并行恢复并行备份

并行恢复并行备份需要来自 NDB 8.0 发行版的**ndb_restore**二进制文件。该过程与在**ndb_restore**程序描述下的一般用法部分中概述的过程没有实质性区别，并且包括执行两次**ndb_restore**，类似于以下所示：

```sql
$> ndb_restore -n 1 -b 1 -m --backup-path=*path/to/backup_dir*/BACKUP/BACKUP-*backup_id*
$> ndb_restore -n 1 -b 1 -r --backup-path=*path/to/backup_dir*/BACKUP/BACKUP-*backup_id*
```

*`backup_id`*是要恢复的备份的 ID。在一般情况下，不需要额外的特殊参数；**ndb_restore**始终检查由`--backup-path`选项指示的目录下是否存在并行子目录，并恢复元数据（串行）然后表数据（并行）。

##### 25.5.23.3.2 串行恢复并行备份

可以以串行方式恢复在数据节点上使用并行方式创建的备份。为此，请调用**ndb_restore**，并使用`--backup-path`指向每个 LDM 在主备份目录下创建的子目录，首先恢复任何一个子目录以恢复元数据（由于每个子目录包含元数据的完整副本，因此无论选择哪一个都可以），然后依次恢复每个子目录中的数据。假设我们想要恢复备份 ID 为 100 的备份，该备份使用四个 LDMs，并且`BackupDataDir`为`/opt`。在这种情况下，我们可以像这样调用**ndb_restore**来恢复元数据：

```sql
$> ndb_restore -n 1 -b 1 -m --backup-path=opt/BACKUP/BACKUP-100/BACKUP-100-PART-1-OF-4
```

要恢复表数据，请依次执行**ndb_restore** 四次，每次使用一个子目录，如下所示：

```sql
$> ndb_restore -n 1 -b 1 -r --backup-path=opt/BACKUP/BACKUP-100/BACKUP-100-PART-1-OF-4
$> ndb_restore -n 1 -b 1 -r --backup-path=opt/BACKUP/BACKUP-100/BACKUP-100-PART-2-OF-4
$> ndb_restore -n 1 -b 1 -r --backup-path=opt/BACKUP/BACKUP-100/BACKUP-100-PART-3-OF-4
$> ndb_restore -n 1 -b 1 -r --backup-path=opt/BACKUP/BACKUP-100/BACKUP-100-PART-4-OF-4
```

您可以使用相同的技术将并行备份恢复到不支持并行备份的较旧版本的 NDB 集群（7.6 或更早版本），使用较旧版本 NDB 集群软件提供的**ndb_restore**二进制文件。
