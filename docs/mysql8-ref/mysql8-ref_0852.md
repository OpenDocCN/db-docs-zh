# 14.18 复制函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-functions.html)

14.18.1 组复制函数

14.18.2 与全局事务标识符（GTID）一起使用的函数

14.18.3 异步复制通道故障转移函数

14.18.4 基于位置的同步函数

下面描述的函数与 MySQL 复制一起使用。

**表 14.24 复制函数**

| 名称 | 描述 | 引入版本 | 废弃版本 |
| --- | --- | --- | --- |
| `asynchronous_connection_failover_add_managed()` | 将组成员源服务器配置信息添加到复制通道源列表 | 8.0.23 |  |
| `asynchronous_connection_failover_add_source()` | 将源服务器配置信息添加到复制通道源列表 | 8.0.22 |  |
| `asynchronous_connection_failover_delete_managed()` | 从复制通道源列表中删除受管理的组 | 8.0.23 |  |
| `asynchronous_connection_failover_delete_source()` | 从复制通道源列表中删除源服务器 | 8.0.22 |  |
| `asynchronous_connection_failover_reset()` | 删除与组复制异步故障转移相关的所有设置 | 8.0.27 |  |
| `group_replication_disable_member_action()` | 禁用指定事件的成员操作 | 8.0.26 |  |
| `group_replication_enable_member_action()` | 启用指定事件的成员操作 | 8.0.26 |  |
| `group_replication_get_communication_protocol()` | 获取当前使用的组复制通信协议版本 | 8.0.16 |  |
| `group_replication_get_write_concurrency()` | 获取当前为组设置的最大一致性实例数 | 8.0.13 |  |
| `group_replication_reset_member_actions()` | 将所有成员操作重置为默认值，并将配置版本号设置为 1 | 8.0.26 |  |
| `group_replication_set_as_primary()` | 将特定组成员设为主节点 | 8.0.29 |  |
| `group_replication_set_communication_protocol()` | 设置用于组复制通信协议的版本 | 8.0.16 |  |
| `group_replication_set_write_concurrency()` | 设置可以并行执行的最大一致性实例数 | 8.0.13 |  |
| `group_replication_switch_to_multi_primary_mode()` | 将运行在单主模式下的组的模式更改为多主模式 | 8.0.13 |  |
| `group_replication_switch_to_single_primary_mode()` | 将运行在多主模式下的组的模式更改为单主模式 | 8.0.13 |  |
| `GTID_SUBSET()` | 如果子集中的所有 GTIDs 也在集合中，则返回 true；否则返回 false。 |  |  |
| `GTID_SUBTRACT()` | 返回集合中不在子集中的所有 GTIDs。 |  |  |
| `MASTER_POS_WAIT()` | 阻塞，直到副本读取并应用到指定位置的所有更新 |  | 8.0.26 |
| `SOURCE_POS_WAIT()` | 阻塞，直到副本读取并应用到指定位置的所有更新 | 8.0.26 |  |
| `WAIT_FOR_EXECUTED_GTID_SET()` | 等待给定的 GTIDs 在副本上执行。 |  |  |
| `WAIT_UNTIL_SQL_THREAD_AFTER_GTIDS()` | 使用`WAIT_FOR_EXECUTED_GTID_SET()`。 |  | 8.0.18 |
| 名称 | 描述 | 引入版本 | 废弃版本 |
