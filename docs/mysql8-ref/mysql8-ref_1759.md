> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cluster-transactions.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cluster-transactions.html)

#### 25.6.16.8 ndbinfo cluster_transactions 表

`cluster_transactions`表显示了 NDB 集群中所有正在进行的事务的信息。

`cluster_transactions`表包含以下列：

+   `node_id`

    事务协调器的节��ID

+   `block_instance`

    TC 块实例

+   `transid`

    事务 ID

+   `state`

    操作状态（请参阅文本以获取可能的值）

+   `count_operations`

    事务中有状态的主键操作数（包括带锁的读取以及 DML 操作）

+   `outstanding_operations`

    仍在本地数据管理块中执行的操作

+   `inactive_seconds`

    等待 API 的时间

+   `client_node_id`

    客户端节点 ID

+   `client_block_ref`

    客户端块引用

##### 注意

事务 ID 是一个唯一的 64 位数字，可以使用 NDB API 的`getTransactionId()`方法获取。 （目前，MySQL 服务器不会公开正在进行的事务的 NDB API 事务 ID。）

`block_instance`指的是内核块的一个实例。与块名称一起，这个数字可以用来在`threadblocks`表中查找给定实例。

`state`列可以有以下任一值：`CS_ABORTING`、`CS_COMMITTING`、`CS_COMMIT_SENT`、`CS_COMPLETE_SENT`、`CS_COMPLETING`、`CS_CONNECTED`、`CS_DISCONNECTED`、`CS_FAIL_ABORTED`、`CS_FAIL_ABORTING`、`CS_FAIL_COMMITTED`、`CS_FAIL_COMMITTING`、`CS_FAIL_COMPLETED`、`CS_FAIL_PREPARED`、`CS_PREPARE_TO_COMMIT`、`CS_RECEIVING`、`CS_REC_COMMITTING`、`CS_RESTART`、`CS_SEND_FIRE_TRIG_REQ`、`CS_STARTED`、`CS_START_COMMITTING`、`CS_START_SCAN`、`CS_WAIT_ABORT_CONF`、`CS_WAIT_COMMIT_CONF`、`CS_WAIT_COMPLETE_CONF`、`CS_WAIT_FIRE_TRIG_REQ`。（如果 MySQL 服务器正在运行时启用了`ndbinfo_show_hidden`，您可以通过从`ndb$dbtc_apiconnect_state`表中选择来查看此状态列表，该表通常是隐藏的。）

在`client_node_id`和`client_block_ref`中，`client`指的是 NDB 集群 API 或 SQL 节点（即，连接到集群的 NDB API 客户端或 MySQL 服务器）。

`tc_block_instance`列提供了`DBTC`块实例号。您可以将此与块名称一起使用，从`threadblocks`表中获取有关特定线程的信息。
