> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-backup-parallel-data-nodes.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-backup-parallel-data-nodes.html)

#### 25.6.8.5 使用并行数据节点进行 NDB 备份

在 NDB 8.0 中，可以使用多个本地数据管理器（LDM）并行在数据节点上进行备份。为了使其工作，集群中的所有数据节点必须使用多个 LDM，并且每个数据节点必须使用相同数量的 LDM。这意味着所有数据节点必须运行**ndbmtd**（**ndbd**是单线程的，因此始终只有一个 LDM）并且在进行备份之前必须配置为使用多个 LDM；**ndbmtd**默认以单线程模式运行。您可以通过选择多线程数据节点配置参数`MaxNoOfExecutionThreads`或`ThreadConfig`的适当设置来使它们使用多个 LDM。请记住，更改这些参数需要重新启动集群；这可以是滚动重启。此外，每个数据节点的`EnableMultithreadedBackup`参数必须设置为 1（这是默认值）。

根据 LDM 的数量和其他因素，您可能还需要增加`NoOfFragmentLogParts`。如果您正在使用大型磁盘数据表，您可能还需要增加`DiskPageBufferMemory`。与单线程备份一样，您可能还希望或需要调整`BackupDataBufferSize`、`BackupMemory`和其他与备份相关的配置参数（参见备份参数）。

一旦所有数据节点都使用多个 LDM，您可以使用 NDB 管理客户端中的`START BACKUP`命令进行并行备份，就像数据节点正在运行**ndbd**（或单线程模式下的**ndbmtd**")）一样；不需要额外或特殊的语法，您可以根据需要或愿望指定备份 ID、等待选项或快照选项的任意组合。

使用多个 LDM 进行备份会在每个数据节点的目录`BACKUP/BACKUP-*`backup_id`*/`（该目录位于`BackupDataDir`下）下创建子目录，这些子目录分别命名为`BACKUP-*`backup_id`*-PART-1-OF-*`N`*/`、`BACKUP-*`backup_id`*-PART-2-OF-*`N`*/`，依此类推，直到`BACKUP-*`backup_id`*-PART-*`N`*-OF-*`N`*/`，其中*`backup_id`*是此备份使用的备份 ID，*`N`*是每个数据节点的 LDM 数。每个子目录包含常规备份文件`BACKUP-*`backup_id`*-0.*`node_id`*.Data`、`BACKUP-*`backup_id`*.*`node_id`*.ctl`和`BACKUP-backup_id.node_id.log`，其中*`node_id`*是此数据节点的节点 ID。

**ndb_restore**会自动检查刚才描述的子目录是否存在；如果找到它们，它将尝试并行恢复备份。有关使用多个 LDM 进行备份的恢复信息，请参阅第 25.5.23.3 节，“从并行备份中恢复”。

要强制创建一个单线程备份，以便可以轻松地由 NDB 8.0 之前的版本的**ndb_restore**导入，您可以为所有数据节点设置`EnableMultithreadedBackup = 0`（您可以通过在`config.ini`全局配置文件的`[ndbd default]`部分中设置该参数来执行此操作）。还可以将并行备份恢复到运行较旧版本`NDB`的集群中。有关更多信息，请参阅第 25.5.23.1.1 节，“将 NDB 备份恢复到较旧版本的 NDB 集群”。
