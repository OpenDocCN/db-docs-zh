> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-arbitrator-validity-summary.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-arbitrator-validity-summary.html)

#### 25.6.16.2 ndbinfo 仲裁者有效性摘要表

`arbitrator_validity_summary` 表提供了关于集群数据节点的仲裁者的综合视图。

`arbitrator_validity_summary` 表包含以下列：

+   `arbitrator`

    仲裁者的节点 ID

+   `arb_ticket`

    用于跟踪仲裁的内部标识符

+   `arb_connected`

    是否这个仲裁者连接到集群

+   `consensus_count`

    视为仲裁者的数据节点数量；可能是`Yes`或`No`

##### 备注

在正常操作中，这个表应该在相当长的时间内只有 1 行。如果在超过几分钟的时间内有多于 1 行，则可能是并非所有节点都连接到仲裁者，或者所有节点都连接了，但对同一个仲裁者意见不一致。

`arbitrator` 列显示仲裁者的节点 ID。

`arb_ticket` 是这个仲裁者使用的内部标识符。

`arb_connected` 显示这个节点是否作为仲裁者连接到集群。
