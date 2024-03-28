# 25.6 NDB 集群管理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-management.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-management.html)

25.6.1 NDB 集群管理客户端中的命令

25.6.2 NDB 集群日志消息

25.6.3 NDB 集群生成的事件报告

25.6.4 NDB 集群启动阶段摘要

25.6.5 对 NDB 集群执行滚动重启

25.6.6 NDB 集群单用户模式

25.6.7 在线添加 NDB 集群数据节点

25.6.8 NDB 集群在线备份

25.6.9 将数据导入 MySQL 集群

25.6.10 用于 NDB 集群的 MySQL 服务器用法

25.6.11 NDB 集群磁盘数据表

25.6.12 在 NDB 集群中使用 ALTER TABLE 进行在线操作

25.6.13 权限同步和 NDB_STORED_USER

25.6.14 NDB 集群的文件系统加密

25.6.15 NDB API 统计计数器和变量

25.6.16 ndbinfo：NDB 集群信息数据库

25.6.17 用于 NDB 集群的 INFORMATION_SCHEMA 表

25.6.18 NDB 集群和性能模式

25.6.19 快速参考：NDB 集群 SQL 语句

25.6.20 NDB 集群安全问题

管理 NDB 集群涉及许多任务，首先是配置和启动 NDB 集群。这在第 25.4 节，“NDB 集群配置”和第 25.5 节，“NDB 集群程序”中有所涵盖。

接下来的几节涵盖了运行 NDB 集群的管理。

有关与 NDB 集群管理和部署相关的安全问题，请参阅第 25.6.20 节，“NDB 集群安全问题”。

实际上有两种主动管理运行中 NDB 集群的方法。其中之一是通过输入到管理客户端的命令来检查集群状态，更改日志级别，启动和停止备份，以及停止和启动节点。第二种方法涉及研究集群日志`ndb_*`node_id`*_cluster.log`的内容；通常可以在管理服务器的`DataDir`目录中找到，但可以使用`LogDestination`选项覆盖此位置。（请记住*`node_id`*代表被记录活动的节点的唯一标识符。）集群日志包含由**ndbd**生成的事件报告。还可以将集群日志条目发送到 Unix 系统日志。

一些集群操作的方面也可以通过在 SQL 节点上使用`SHOW ENGINE NDB STATUS`语句来监控。

通过使用`ndbinfo`数据库，可以实时通过 SQL 接口获取有关 NDB 集群操作的更详细信息。有关更多信息，请参见 Section 25.6.16, “ndbinfo: The NDB Cluster Information Database”。

NDB 统计计数器通过**mysql**客户端提供了改进的监控。这些计数器，由 NDB 内核实现，涉及由或影响`Ndb`对象执行的操作，如启动、关闭和中止事务；主键和唯一键操作；表、范围和修剪扫描；等待各种操作完成的阻塞线程；以及 NDB 集群发送和接收的数据和事件。每当进行 NDB API 调用或数据被发送到或接收到数据节点时，NDB 内核都会递增计数器。

**mysqld**将 NDB API 统计计数器公开为系统状态变量，可以从它们的名称中共同前缀（`Ndb_api_`）中识别。这些变量的值可以通过`SHOW STATUS`语句的输出在**mysql**客户端中读取，或者通过查询 Performance Schema 的`session_status`或`global_status`表。通过比较执行对`NDB`表操作的 SQL 语句之前和之后的状态变量的值，您可以观察到与该语句对应的 NDB API 级别的操作，这对于监视和性能调整 NDB Cluster 可能是有益的。

MySQL Cluster Manager 提供了一个高级命令行界面，简化了许多复杂的 NDB Cluster 管理任务，比如启动、停止或重新启动具有大量节点的 NDB Cluster。MySQL Cluster Manager 客户端还支持获取和设置大多数节点配置参数以及与 NDB Cluster 相关的**mysqld**服务器选项和变量的命令。MySQL Cluster Manager 8.0 支持 NDB 8.0。有关更多信息，请参阅 MySQL Cluster Manager 8.0.36 用户手册。
