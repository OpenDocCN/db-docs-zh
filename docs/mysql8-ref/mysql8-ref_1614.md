# 第二十五章 MySQL NDB 集群 8.0

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster.html)

**目录**

25.1 通用信息

25.2 NDB 集群概述

25.2.1 NDB 集群核心概念

25.2.2 NDB 集群节点、节点组、片段副本和分区

25.2.3 NDB 集群的硬件、软件和网络要求

25.2.4 MySQL NDB 集群 8.0 中的新功能

25.2.5 NDB 8.0 中新增、弃用或移除的选项、变量和参数

25.2.6 使用 InnoDB 的 MySQL 服务器与 NDB 集群的比较

25.2.7 NDB 集群的已知限制

25.3 NDB 集群安装

25.3.1 在 Linux 上安装 NDB 集群

25.3.2 在 Windows 上安装 NDB 集群

25.3.3 NDB 集群的初始配置

25.3.4 NDB 集群的初始启动

25.3.5 具有表和数据的 NDB 集群示例

25.3.6 NDB 集群的安全关闭和重启

25.3.7 NDB 集群的升级和降级

25.3.8 NDB 集群自动安装程序（不再支持）

25.4 NDB 集群的配置

25.4.1 NDB 集群的快速测试设置

25.4.2 NDB 集群配置参数、选项和变量概述

25.4.3 NDB 集群配置文件

25.4.4 使用高速互连与 NDB 集群

25.5 NDB 集群程序

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

25.5.24 ndb_secretsfile_reader — 从加密的 NDB 数据文件中获取关键信息

25.5.25 ndb_select_all — 打印 NDB 表中的行

25.5.26 ndb_select_count — 打印 NDB 表的行计数

25.5.27 ndb_show_tables — 显示 NDB 表列表

25.5.28 ndb_size.pl — NDBCLUSTER 大小需求估算器

25.5.29 ndb_top — 查看 NDB 线程的 CPU 使用信息

25.5.30 ndb_waiter — 等待 NDB 集群达到给定状态

25.5.31 ndbxfrm — 压缩、解压、加密和解密由 NDB 集群创建的文件

25.6 NDB 集群的管理

25.6.1 NDB 集群管理客户端中的命令

25.6.2 NDB 集群日志消息

25.6.3 在 NDB 集群中生成的事件报告

25.6.4 NDB 集群启动阶段总结

25.6.5 执行 NDB Cluster 的滚动重启

25.6.6 NDB Cluster 单用户模式

25.6.7 在线添加 NDB Cluster 数据节点

25.6.8 NDB Cluster 的在线备份

25.6.9 将数据导入到 MySQL Cluster

25.6.10 NDB Cluster 的 MySQL 服务器用法

25.6.11 NDB Cluster 磁盘数据表

25.6.12 NDB Cluster 中使用 ALTER TABLE 的在线操作

25.6.13 权限同步和 NDB_STORED_USER

25.6.14 NDB Cluster 的文件系统加密

25.6.15 NDB API 统计计数器和变量

25.6.16 ndbinfo：NDB Cluster 信息数据库

25.6.17 NDB Cluster 的 INFORMATION_SCHEMA 表

25.6.18 NDB Cluster 和性能模式

25.6.19 快速参考：NDB Cluster SQL 语句

25.6.20 NDB Cluster 安全问题

25.7 NDB Cluster 复制

25.7.1 NDB Cluster 复制：缩写和符号

25.7.2 NDB Cluster 复制的一般要求

25.7.3 NDB Cluster 复制中已知问题

25.7.4 NDB Cluster 复制模式和表

25.7.5 为复制准备 NDB Cluster

25.7.6 启动 NDB Cluster 复制（单一复制通道）

25.7.7 使用两个复制通道进行 NDB Cluster 复制

25.7.8 使用 NDB Cluster 复制实现故障切换

25.7.9 使用 NDB Cluster 复制进行 NDB Cluster 备份

25.7.10 NDB Cluster 复制：双向和循环复制

25.7.11 使用多线程应用程序的 NDB Cluster 复制

25.7.12 NDB Cluster 复制冲突解决

25.8 NDB Cluster 发行说明

本章提供关于 MySQL NDB Cluster 的信息，这是 MySQL 的高可用性、高冗余性版本，适用于分布式计算环境，使用`NDB`存储引擎（也称为`NDBCLUSTER`）来实现在集群中运行多台带有 MySQL 服务器和其他软件的计算机。NDB Cluster 8.0，现已作为通用可用性（GA）版本发布（从版本 8.0.19 开始），包含了`NDB`存储引擎的 8.0 版本。本章还提供了特定于 NDB Cluster 8.0 的信息，并覆盖了 8.0.34 版本。基于 NDB 存储引擎的 NDB Cluster 8.1（NDB 8.1.0）现已作为创新版本发布。请参阅 MySQL NDB Cluster 8.2 有什么新功能，了解 NDB 8.1 与早期版本的区别。NDB Cluster 7.6 和 NDB Cluster 7.5 仍作为 GA 版本提供，分别使用了 7.6 和 7.5 版本的`NDB`。NDB Cluster 7.6 和 7.5 是之前的 GA 版本，仍在生产中得到支持；有关 NDB Cluster 7.6 的信息，请参阅 NDB Cluster 7.6 有什么新功能。有关 NDB Cluster 7.5 的类似信息，请参阅 NDB Cluster 7.5 有什么新功能。之前的 GA 版本 NDB Cluster 7.4 和 NDB Cluster 7.3 分别包含了`NDB`的 7.4 和 7.3 版本。*NDB 7.4 及更早版本系列不再得到支持或维护*。NDB 8.0 和 NDB 8.1 都在生产中得到支持，并建议用于新部署。
