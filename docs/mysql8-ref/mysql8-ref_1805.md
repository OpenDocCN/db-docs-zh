> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-server-operations.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-server-operations.html)

#### 25.6.16.54 ndbinfo server_operations 表

`server_operations`表包含当前 SQL 节点（MySQL 服务器）当前参与的所有进行中的`NDB`操作的条目。它实际上是`cluster_operations`表的一个子集，其中不显示其他 SQL 和 API 节点的操作。

`server_operations`表包含以下列：

+   `mysql_connection_id`

    MySQL 服务器连接 ID

+   `node_id`

    节点 ID

+   `block_instance`

    块实例

+   `transid`

    事务 ID

+   `operation_type`

    操作类型（请参阅文本以获取可能的值）

+   `state`

    操作状态（请参阅文本以获取可能的值）

+   `tableid`

    表 ID

+   `fragmentid`

    片段 ID

+   `client_node_id`

    客户端节点 ID

+   `client_block_ref`

    客户端块引用

+   `tc_node_id`

    事务协调器节点 ID

+   `tc_block_no`

    事务协调器块编号

+   `tc_block_instance`

    事务协调器块实例

##### 备注

`mysql_connection_id`与`SHOW PROCESSLIST`输出中显示的连接或会话 ID 相同。它从`INFORMATION_SCHEMA`表`NDB_TRANSID_MYSQL_CONNECTION_MAP`中获取。

`block_instance`指的是内核块的一个实例。与块名称一起，此数字可用于在`threadblocks`表中查找给定实例。

事务 ID（`transid`）是一个唯一的 64 位数字，可以使用 NDB API 的`getTransactionId()`方法获取。（目前，MySQL 服务器不公开正在进行的事务的 NDB API 事务 ID。）

`operation_type`列可以取以下任一值：`READ`、`READ-SH`、`READ-EX`、`INSERT`、`UPDATE`、`DELETE`、`WRITE`、`UNLOCK`、`REFRESH`、`SCAN`、`SCAN-SH`、`SCAN-EX`或`<unknown>`。

`state`列可以具有以下任一值：`ABORT_QUEUED`、`ABORT_STOPPED`、`COMMITTED`、`COMMIT_QUEUED`、`COMMIT_STOPPED`、`COPY_CLOSE_STOPPED`、`COPY_FIRST_STOPPED`、`COPY_STOPPED`、`COPY_TUPKEY`、`IDLE`、`LOG_ABORT_QUEUED`、`LOG_COMMIT_QUEUED`、`LOG_COMMIT_QUEUED_WAIT_SIGNAL`、`LOG_COMMIT_WRITTEN`、`LOG_COMMIT_WRITTEN_WAIT_SIGNAL`、`LOG_QUEUED`、`PREPARED`、`PREPARED_RECEIVED_COMMIT`、`SCAN_CHECK_STOPPED`、`SCAN_CLOSE_STOPPED`、`SCAN_FIRST_STOPPED`、`SCAN_RELEASE_STOPPED`、`SCAN_STATE_USED`、`SCAN_STOPPED`、`SCAN_TUPKEY`、`STOPPED`、`TC_NOT_CONNECTED`、`WAIT_ACC`、`WAIT_ACC_ABORT`、`WAIT_AI_AFTER_ABORT`、`WAIT_ATTR`、`WAIT_SCAN_AI`、`WAIT_TUP`、`WAIT_TUPKEYINFO`、`WAIT_TUP_COMMIT`或`WAIT_TUP_TO_ABORT`。（如果 MySQL 服务器使用`ndbinfo_show_hidden`启用运行，则可以通过从通常隐藏的`ndb$dblqh_tcconnect_state`表中选择来查看此状态列表。）

通过检查**ndb_show_tables**的输出，您可以通过表 ID 获取`NDB`表的名称。

`fragid`与**ndb_desc**的输出中看到的分区号相同，`--extra-partition-info`（简写为`-p`）。

在`client_node_id`和`client_block_ref`中，`client`指的是 NDB Cluster API 或 SQL 节点（即，NDB API 客户端或连接到集群的 MySQL 服务器）。

`block_instance`和`tc_block_instance`列提供 NDB 内核块实例号。您可以使用这些信息从`threadblocks`表中获取有关特定线程的信息。
