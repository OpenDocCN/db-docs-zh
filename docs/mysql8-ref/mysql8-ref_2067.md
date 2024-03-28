# 29.12.12 性能模式 NDB 集群表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-ndb-cluster-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-ndb-cluster-tables.html)

29.12.12.1 ndb_sync_pending_objects 表

29.12.12.2 ndb_sync_excluded_objects 表

以下表格显示了所有与`NDBCLUSTER`存储引擎相关的性能模式表。

**表 29.3 性能模式 NDB 表**

| 表名 | 描述 | 引入版本 |
| --- | --- | --- |
| `ndb_sync_excluded_objects` | 无法同步的 NDB 对象 | 8.0.21 |
| `ndb_sync_pending_objects` | 等待同步的 NDB 对象 | 8.0.21 |

从 NDB 8.0.16 开始，`NDB` 中的自动同步尝试检测并自动同步 NDB 集群内部字典与 MySQL 服务器数据字典之间的所有元数据不匹配。默认情况下，这是通过后台定期间隔（由 `ndb_metadata_check_interval` 系统变量确定）完成的，除非使用 `ndb_metadata_check` 禁用或通过设置 `ndb_metadata_sync` 覆盖。在 NDB 8.0.21 之前，用户可以获得关于此过程的唯一信息是以日志消息和对象计数的形式提供的（从 NDB 8.0.18 开始可用），作为状态变量 `Ndb_metadata_detected_count`、`Ndb_metadata_synced_count` 和 `Ndb_metadata_excluded_count`（在 NDB 8.0.22 之前，此变量名为 `Ndb_metadata_blacklist_size`）。从 NDB 8.0.21 开始，作为 NDB 集群中的 SQL 节点充当的 MySQL 服务器在这两个性能模式表中公开有关自动同步当前状态的更详细信息：

+   `ndb_sync_pending_objects`：显示了在`NDB`字典和 MySQL 数据字典之间检测到不匹配的`NDB`数据库对象的信息。当尝试同步这些对象时，`NDB`会将对象从等待同步的队列中移除，并从此表中移除，并尝试协调不匹配。如果由于临时错误导致对象同步失败，则会在下次`NDB`执行不匹配检测时重新添加到队列（和此表）；如果尝试由于永久错误而失败，则将对象添加到`ndb_sync_excluded_objects`表中。

+   `ndb_sync_excluded_objects`：显示了由于不可调和的不匹配导致自动同步失败的`NDB`数据库对象的信息；这些对象被列入黑名单，并且在进行手动干预之前不再考虑进行不匹配检测。

只有当 MySQL 启用了对`NDBCLUSTER`存储引擎的支持时，`ndb_sync_pending_objects`和`ndb_sync_excluded_objects`表才存在。

这些表在以下两个部分中有更详细的描述。
