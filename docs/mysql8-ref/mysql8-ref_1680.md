# 25.5 NDB 集群程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs.html)

25.5.1 ndbd — NDB 集群数据节点守护程序

25.5.2 ndbinfo_select_all — 从 ndbinfo 表中选择

25.5.3 ndbmtd — NDB 集群数据节点守护程序（多线程）

25.5.4 ndb_mgmd — NDB 集群管理服务器守护程序

25.5.5 ndb_mgm — NDB 集群管理客户端

25.5.6 ndb_blob_tool — 检查和修复 NDB 集群表的 BLOB 和 TEXT 列

25.5.7 ndb_config — 提取 NDB 集群配置信息

25.5.8 ndb_delete_all — 从 NDB 表中删除所有行

25.5.9 ndb_desc — 描述 NDB 表

25.5.10 ndb_drop_index — 从 NDB 表中删除索引

25.5.11 ndb_drop_table — 删除 NDB 表

25.5.12 ndb_error_reporter — NDB 错误报告实用程序

25.5.13 ndb_import — 将 CSV 数据导入 NDB

25.5.14 ndb_index_stat — NDB 索引统计实用程序

25.5.15 ndb_move_data — NDB 数据复制实用程序

25.5.16 ndb_perror — 获取 NDB 错误消息信息

25.5.17 ndb_print_backup_file — 打印 NDB 备份文件内容

25.5.18 ndb_print_file — 打印 NDB 磁盘数据文件内容

25.5.19 ndb_print_frag_file — 打印 NDB 片段列表文件内容

25.5.20 ndb_print_schema_file — 打印 NDB 模式文件内容

25.5.21 ndb_print_sys_file — 打印 NDB 系统文件内容

25.5.22 ndb_redo_log_reader — 检查和打印集群重做日志的内容

25.5.23 ndb_restore — 恢复 NDB 集群备份

25.5.24 ndb_secretsfile_reader — 从加密的 NDB 数据文件中获取密钥信息

25.5.25 ndb_select_all — 打印 NDB 表中的行

25.5.26 ndb_select_count — 打印 NDB 表的行数

25.5.27 ndb_show_tables — 显示 NDB 表的列表

25.5.28 ndb_size.pl — NDBCLUSTER 大小需求估算器

25.5.29 ndb_top — 查看 NDB 线程的 CPU 使用信息

25.5.30 ndb_waiter — 等待 NDB 集群达到给定状态

25.5.31 ndbxfrm — 压缩、解压、加密和解密由 NDB 集群创建的文件

使用和管理 NDB 集群需要几个专门的程序，在本章中我们将对这些程序进行描述。我们讨论这些程序在 NDB 集群中的目的，如何使用这些程序以及每个程序可用的启动选项。

这些程序包括 NDB 集群数据、管理和 SQL 节点进程（**ndbd**, **ndbmtd**, **ndb_mgmd**, 和 **mysqld**) 以及管理客户端（**ndb_mgm**）。

有关将**mysqld**用作 NDB 集群进程的信息，请参阅第 25.6.10 节，“NDB 集群的 MySQL 服务器用法”。

其他`NDB` 实用程序、诊断和示例程序包含在 NDB 集群分发中。这些包括 **ndb_restore**, **ndb_show_tables**, 和 **ndb_config**. 这些程序也在本节中介绍。
