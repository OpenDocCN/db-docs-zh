> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-server-transactions.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-server-transactions.html)

#### 25.6.16.55 ndbinfo server_transactions 表

`server_transactions`表是`cluster_transactions`表的子集，但仅包括当前 SQL 节点（MySQL 服务器）参与的事务，并包括相关的连接 ID。

`server_transactions`表包含以下列：

+   `mysql_connection_id`

    MySQL 服务器连接 ID

+   `node_id`

    事务协调器节点 ID

+   `block_instance`

    事务协调器块实例

+   `transid`

    事务 ID

+   `state`

    操作状态（请参阅可能的值）

+   `count_operations`

    事务中有状态的操作数

+   `outstanding_operations`

    仍由本地数据管理层（LQH 块）执行的操作

+   `inactive_seconds`

    等待 API 的时间

+   `client_node_id`

    客户端节点 ID

+   `client_block_ref`

    客户端块引用

##### 注意事项

`mysql_connection_id`与`SHOW PROCESSLIST`输出中显示的连接或会话 ID 相同。它从`INFORMATION_SCHEMA`表`NDB_TRANSID_MYSQL_CONNECTION_MAP`中获取。

`block_instance`指的是内核块的一个实例。与块名称一起，此数字可用于在`threadblocks`表中查找给定实例。

事务 ID（`transid`）是一个唯一的 64 位数字，可以使用 NDB API 的`getTransactionId()`方法获取。（目前，MySQL 服务器不公开正在进行的事务的 NDB API 事务 ID。）

`state`列可以具有以下任一值：`CS_ABORTING`、`CS_COMMITTING`、`CS_COMMIT_SENT`、`CS_COMPLETE_SENT`、`CS_COMPLETING`、`CS_CONNECTED`、`CS_DISCONNECTED`、`CS_FAIL_ABORTED`、`CS_FAIL_ABORTING`、`CS_FAIL_COMMITTED`、`CS_FAIL_COMMITTING`、`CS_FAIL_COMPLETED`、`CS_FAIL_PREPARED`、`CS_PREPARE_TO_COMMIT`、`CS_RECEIVING`、`CS_REC_COMMITTING`、`CS_RESTART`、`CS_SEND_FIRE_TRIG_REQ`、`CS_STARTED`、`CS_START_COMMITTING`、`CS_START_SCAN`、`CS_WAIT_ABORT_CONF`、`CS_WAIT_COMMIT_CONF`、`CS_WAIT_COMPLETE_CONF`、`CS_WAIT_FIRE_TRIG_REQ`。（如果 MySQL 服务器使用`ndbinfo_show_hidden`启用运行，则可以通过从通常隐藏的`ndb$dbtc_apiconnect_state`表中选择来查看此状态列表。）

在`client_node_id`和`client_block_ref`中，`client`指的是 NDB 集群 API 或 SQL 节点（即，NDB API 客户端或连接到集群的 MySQL 服务器）。

`block_instance`列提供了`DBTC`内核块实例编号。您可以使用此信息从`threadblocks`表中获取有关特定线程的信息。
