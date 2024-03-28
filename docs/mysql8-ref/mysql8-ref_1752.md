> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-arbitrator-validity-detail.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-arbitrator-validity-detail.html)

#### 25.6.16.1 `ndbinfo arbitrator_validity_detail` 表

`arbitrator_validity_detail` 表显示集群中每个数据节点对仲裁者的视图。它是`membership`表的子集。

`arbitrator_validity_detail` 表包含以下列：

+   `node_id`

    此节点的节点 ID

+   `仲裁者`

    仲裁者的节点 ID

+   `arb_ticket`

    用于跟踪仲裁的内部标识符

+   `arb_connected`

    此节点是否连接到仲裁者；是`Yes`或`No`中的一个

+   `arb_state`

    仲裁状态

##### 备注

节点 ID 与**ndb_mgm -e "SHOW"**报告的相同。

所有节点应该显示相同的`仲裁者`和`arb_ticket`值，以及相同的`arb_state`值。可能的`arb_state`值包括`ARBIT_NULL`、`ARBIT_INIT`、`ARBIT_FIND`、`ARBIT_PREP1`、`ARBIT_PREP2`、`ARBIT_START`、`ARBIT_RUN`、`ARBIT_CHOOSE`、`ARBIT_CRASH`和`UNKNOWN`。

`arb_connected` 显示当前节点是否连接到`仲裁者`。
