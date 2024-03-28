# 25.5.20 ndb_print_schema_file — 打印 NDB 模式文件内容

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-print-schema-file.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-print-schema-file.html)

**ndb_print_schema_file**从集群模式文件获取诊断信息。

#### 用法

```sql
ndb_print_schema_file *file_name*
```

*`file_name`* 是一个集群模式文件的名称。有关集群模式文件的更多信息，请参阅 NDB Cluster Data Node File System Directory。

像**ndb_print_backup_file**和**ndb_print_sys_file**（与大多数旨在在管理服务器主机上运行或连接到管理服务器的其他`NDB`实用程序不同）**ndb_print_schema_file**必须在集群数据节点上运行，因为它直接访问数据节点文件系统。由于它不使用管理服务器，因此此实用程序可在管理服务器未运行时使用，甚至在集群完全关闭时也可使用。

#### 附加选项

无。
