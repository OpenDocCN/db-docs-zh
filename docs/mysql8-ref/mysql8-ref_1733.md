# 25.6.8 NDB 集群在线备份

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-backup.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-backup.html)

25.6.8.1 NDB 集群备份概念

25.6.8.2 使用 NDB 集群管理客户端创建备份

25.6.8.3 NDB 集群备份配置

25.6.8.4 NDB 集群备份故障排除

25.6.8.5 使用并行数据节点进行 NDB 备份

接下来的几节描述了如何准备并创建 NDB 集群备份，使用专门用于此目的的 **ndb_mgm** 管理客户端的功能。为了区分这种类型的备份与使用 **mysqldump** 创建的备份，我们有时将其称为“本地” NDB 集群备份。（有关使用 **mysqldump** 创建备份的信息，请参见 第 6.5.4 节，“mysqldump — 数据库备份程序”。）NDB 集群备份的恢复使用 NDB 集群分发提供的 **ndb_restore** 实用程序完成；有关 **ndb_restore** 及其在恢复 NDB 集群备份中的使用的信息，请参见 第 25.5.23 节，“ndb_restore — 恢复 NDB 集群备份”。

NDB 8.0 可以使用多个 LDM 来创建备份，以实现数据节点的并行性。参见 第 25.6.8.5 节，“使用并行数据节点进行 NDB 备份”。
