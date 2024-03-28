> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-configuration-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-applier-configuration-table.html)

#### 29.12.11.5 复制 _applier_configuration 表

此表显示影响副本应用的事务的配置参数。表中存储的参数可以使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）在运行时更改。

`replication_applier_configuration` 表具有以下列：

+   `CHANNEL_NAME`

    此行显示的复制通道。始终存在默认复制通道，并且可以添加更多复制通道。有关更多信息，请参见第 19.2.2 节，“复制通道”。

+   `DESIRED_DELAY`

    复制品必须滞后源的秒数。 (`CHANGE REPLICATION SOURCE TO` 选项：`SOURCE_DELAY`，`CHANGE MASTER TO` 选项：`MASTER_DELAY`)。有关更多信息，请参见第 19.4.11 节，“延迟复制”。

+   `PRIVILEGE_CHECKS_USER`

    为通道提供安全上下文的用户帐户（`CHANGE REPLICATION SOURCE TO` 选项：`PRIVILEGE_CHECKS_USER`，`CHANGE MASTER TO` 选项：`PRIVILEGE_CHECKS_USER`）。这是经过转义的，以便可以将其复制到 SQL 语句中以执行单个事务。有关更多信息，请参见第 19.3.3 节，“复制权限检查”。

+   `REQUIRE_ROW_FORMAT`

    通道是否仅接受基于行的事件（`CHANGE REPLICATION SOURCE TO` 选项：`REQUIRE_ROW_FORMAT`，`CHANGE MASTER TO` 选项：`REQUIRE_ROW_FORMAT`）。有关更多信息，请参见第 19.3.3 节，“复制权限检查”。

+   `REQUIRE_TABLE_PRIMARY_KEY_CHECK`

    通道是否始终需要主键，从不需要，或根据源的设置（`CHANGE REPLICATION SOURCE TO` 选项：`REQUIRE_TABLE_PRIMARY_KEY_CHECK`，`CHANGE MASTER TO` 选项：`REQUIRE_TABLE_PRIMARY_KEY_CHECK`）。有关更多信息，请参见第 19.3.3 节，“复制权限检查”。

+   `ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS_TYPE`

    通道是否为尚未具有 GTID 的复制事务分配 GTID（`CHANGE REPLICATION SOURCE TO` 选项：`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`，`CHANGE MASTER TO` 选项：`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`）。`OFF` 表示不分配 GTID。`LOCAL` 表示分配包含副本自身 UUID（`server_uuid` 设置）的 GTID。`UUID` 表示分配包含手动设置 UUID 的 GTID。更多信息请参见 Section 19.1.3.6, “Replication From a Source Without GTIDs to a Replica With GTIDs”。

+   `ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS_VALUE`

    用作分配给匿名事务的 GTIDs 的 UUID（`CHANGE REPLICATION SOURCE TO` 选项：`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`，`CHANGE MASTER TO` 选项：`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`）。更多信息请参见 Section 19.1.3.6, “Replication From a Source Without GTIDs to a Replica With GTIDs”。

`replication_applier_configuration` 表具有以下索引：

+   主键为 (`CHANNEL_NAME`)

`TRUNCATE TABLE` 不允许用于 `replication_applier_configuration` 表。

以下表格显示了 `replication_applier_configuration` 列与 `SHOW REPLICA STATUS` 列之间的对应关系。

| `replication_applier_configuration` 列 | `SHOW REPLICA STATUS` 列 |
| --- | --- |
| `DESIRED_DELAY` | `SQL_Delay` |
