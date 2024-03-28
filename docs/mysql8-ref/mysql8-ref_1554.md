> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-compatibility-communication.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-compatibility-communication.html)

#### 20.8.1.2 Group Replication 通信协议版本

复制组使用的 Group Replication 通信协议版本可能与成员的 MySQL Server 版本不同。要检查组的通信协议版本，请在任何成员上发出以下语句：

```sql
SELECT group_replication_get_communication_protocol();
```

返回值显示了可以加入此组并使用组通信协议的最旧 MySQL Server 版本。从 MySQL 5.7.14 版本开始允许消息压缩，而从 MySQL 8.0.16 版本开始还允许消息分段。请注意，`group_replication_get_communication_protocol()` 函数返回组支持的最低 MySQL 版本，这可能与传递给 `group_replication_set_communication_protocol()` 函数的版本号不同，并且可能与在使用该函数的成员上安装的 MySQL Server 版本不同。

当您将复制组的所有成员升级到新的 MySQL Server 版本时，Group Replication 通信协议版本不会自动升级，以防仍然需要允许早期版本的成员加入。如果您不需要支持旧版本成员，并希望允许升级后的成员使用任何新增的通信功能，在升级后使用 `group_replication_set_communication_protocol()` 函数升级通信协议，指定您已将成员升级到的新 MySQL Server 版本。有关更多信息，请参见 Section 20.5.1.4, “Setting a Group's Communication Protocol Version”。
