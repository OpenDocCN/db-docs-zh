# 25.5.21 ndb_print_sys_file — 打印 NDB 系统文件内容

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-print-sys-file.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-print-sys-file.html)

**ndb_print_sys_file**从 NDB 集群系统文件中获取诊断信息。

#### 用法

```sql
ndb_print_sys_file *file_name*
```

*`file_name`*是集群系统文件（sysfile）的名称。集群系统文件位于数据节点的数据目录（`DataDir`）中；此目录下系统文件的路径与模式`ndb_*`#`*_fs/D*`#`*/DBDIH/P*`#`*.sysfile`匹配。在每种情况下，*`#`*代表一个数字（不一定是相同的数字）。有关更多信息，请参见 NDB 集群数据节点文件系统目录。

像**ndb_print_backup_file**和**ndb_print_schema_file**（与大多数旨在在管理服务器主机上运行或连接到管理服务器的其他`NDB`实用程序不同）**ndb_print_backup_file**必须在集群数据节点上运行，因为它直接访问数据节点文件系统。由于它不使用管理服务器，因此此实用程序可在管理服务器未运行时使用，甚至在集群完全关闭时也可使用。

#### 附加选项

无。
