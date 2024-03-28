> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-group-write-consensus.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-group-write-consensus.html)

#### 20.5.1.3 使用 Group Replication 群组写共识

本节介绍如何检查和配置群组在任何时候的最大共识实例数。这个最大值被称为群组的事件视界，是群组可以并行执行的最大共识实例数。这使您能够微调您的 Group Replication 部署的性能。例如，默认值为 10 适用于在局域网上运行的群组，但对于在较慢的网络（如广域网）上运行的群组，增加此数字以提高性能。

##### 检查群组的写并发性

使用`group_replication_get_write_concurrency()`函数，在运行时检查群组的事件视界值，通过发出：

```sql
SELECT group_replication_get_write_concurrency();
```

##### 配置群组的写并发性

使用`group_replication_set_write_concurrency()`函数，设置系统可以并行执行的最大共识实例数，通过发出：

```sql
SELECT group_replication_set_write_concurrency(*instances*);
```

其中*`instances`*是新的最大共识实例数。需要`GROUP_REPLICATION_ADMIN`权限才能使用此功能。
