> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-configuration-version-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-configuration-version-table.html)

#### 29.12.11.14 复制组配置版本表

这个表显示了复制组成员操作配置的版本。只有在安装了组复制时才可用。每当使用`group_replication_enable_member_action()`和`group_replication_disable_member_action()`函数启用或禁用成员操作时，版本号会递增。您可以使用`group_replication_reset_member_actions()`函数重置成员操作配置，将其重置为默认设置，并将其版本号重置为 1。更多信息，请参见 Section 20.5.1.5, “Configuring Member Actions”。

`replication_group_configuration_version`表具有以下列：

+   `名称`

    配置的名称。

+   `版本`

    配置的版本号。

`replication_group_configuration_version`表没有索引。

对于`replication_group_configuration_version`表，不允许使用`TRUNCATE TABLE`。
