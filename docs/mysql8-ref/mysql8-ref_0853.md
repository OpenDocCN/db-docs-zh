# 14.18.1 组复制函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-functions.html)

14.18.1.1 配置组复制主要成员的函数

14.18.1.2 配置组复制模式的函数

14.18.1.3 检查和配置组的最大一致性实例数的函数

14.18.1.4 检查和设置组复制通信协议版本的函数

14.18.1.5 设置和重置组复制成员操作的函数

以下部分描述的函数与组复制一起使用。

**表 14.25 组复制函数**

| 名称 | 描述 | 引入版本 |
| --- | --- | --- |
| `group_replication_disable_member_action()` | 禁用指定事件的成员操作 | 8.0.26 |
| `group_replication_enable_member_action()` | 启用指定事件的成员操作 | 8.0.26 |
| `group_replication_get_communication_protocol()` | 获取当前使用的组复制通信协议版本 | 8.0.16 |
| `group_replication_get_write_concurrency()` | 获取当前为组设置的最大一致性实例数 | 8.0.13 |
| `group_replication_reset_member_actions()` | 将所有成员操作重置为默认值，并将配置版本号设置为 1 | 8.0.26 |
| `group_replication_set_as_primary()` | 将特定组成员设为主要成员 | 8.0.29 |
| `group_replication_set_communication_protocol()` | 设置要使用的组复制通信协议版本 | 8.0.16 |
| `group_replication_set_write_concurrency()` | 设置可以并行执行的最大一致性实例数 | 8.0.13 |
| `group_replication_switch_to_multi_primary_mode()` | 将运行在单主模式下的组的模式更改为多主模式 | 8.0.13 |
| `group_replication_switch_to_single_primary_mode()` | 将运行在多主模式下的组的模式更改为单主模式 | 8.0.13 |
| 名称 | 描述 | 引入版本 |
