# 25.5.19 ndb_print_frag_file — 打印 NDB 片段列表文件内容

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-print-frag-file.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-print-frag-file.html)

**ndb_print_frag_file** 从集群片段列表文件中获取信息。它旨在帮助诊断数据节点重新启动时出现的问题。

#### 用法

```sql
ndb_print_frag_file *file_name*
```

*`file_name`* 是集群片段列表文件的名称，与模式 `S*`X`*.FragList` 匹配，其中 *`X`* 是范围在 2-9 之间的数字，并且位于具有节点 ID *`nodeid`* 的数据节点文件系统中，位于名为 `ndb_*`nodeid`*_fs/D*`N`*/DBDIH/` 的目录中，其中 *`N`* 是 `1` 或 `2`。每个片段文件包含属于每个 `NDB` 表的片段记录。有关集群片段文件的更多信息，请参见 NDB 集群数据节点文件系统目录。

与 **ndb_print_backup_file**, **ndb_print_sys_file**, 以及 **ndb_print_schema_file**（与大多数旨在在管理服务器主机上运行或连接到管理服务器的其他 `NDB` 实用程序不同），**ndb_print_frag_file** 必须在集群数据节点上运行，因为它直接访问数据节点文件系统。由于它不使用管理服务器，因此此实用程序可在管理服务器未运行时使用，甚至在集群完全关闭时使用。

#### 附加选项

无。

#### 示例输出

```sql
$> ndb_print_frag_file /usr/local/mysqld/data/ndb_3_fs/D1/DBDIH/S2.FragList
Filename: /usr/local/mysqld/data/ndb_3_fs/D1/DBDIH/S2.FragList with size 8192
noOfPages = 1 noOfWords = 182
Table Data
----------
Num Frags: 2 NoOfReplicas: 2 hashpointer: 4294967040
kvalue: 6 mask: 0x00000000 method: HashMap
Storage is on Logged and checkpointed, survives SR
------ Fragment with FragId: 0 --------
Preferred Primary: 2 numStoredReplicas: 2 numOldStoredReplicas: 0 distKey: 0 LogPartId: 0
-------Stored Replica----------
Replica node is: 2 initialGci: 2 numCrashedReplicas = 0 nextLcpNo = 1
LcpNo[0]: maxGciCompleted: 1 maxGciStarted: 2 lcpId: 1 lcpStatus: valid
LcpNo[1]: maxGciCompleted: 0 maxGciStarted: 0 lcpId: 0 lcpStatus: invalid
-------Stored Replica----------
Replica node is: 3 initialGci: 2 numCrashedReplicas = 0 nextLcpNo = 1
LcpNo[0]: maxGciCompleted: 1 maxGciStarted: 2 lcpId: 1 lcpStatus: valid
LcpNo[1]: maxGciCompleted: 0 maxGciStarted: 0 lcpId: 0 lcpStatus: invalid
------ Fragment with FragId: 1 --------
Preferred Primary: 3 numStoredReplicas: 2 numOldStoredReplicas: 0 distKey: 0 LogPartId: 1
-------Stored Replica----------
Replica node is: 3 initialGci: 2 numCrashedReplicas = 0 nextLcpNo = 1
LcpNo[0]: maxGciCompleted: 1 maxGciStarted: 2 lcpId: 1 lcpStatus: valid
LcpNo[1]: maxGciCompleted: 0 maxGciStarted: 0 lcpId: 0 lcpStatus: invalid
-------Stored Replica----------
Replica node is: 2 initialGci: 2 numCrashedReplicas = 0 nextLcpNo = 1
LcpNo[0]: maxGciCompleted: 1 maxGciStarted: 2 lcpId: 1 lcpStatus: valid
LcpNo[1]: maxGciCompleted: 0 maxGciStarted: 0 lcpId: 0 lcpStatus: invalid
```
