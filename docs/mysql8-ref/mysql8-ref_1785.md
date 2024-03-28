> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-events.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-events.html)

#### 25.6.16.34 ndbinfo 事件表

此表提供有关`NDB`中事件订阅的信息。`events`表的列在此处列出，并附有每个列的简短描述：

+   `event_id`

    事件 ID

+   `name`

    事件的名称

+   `table_id`

    事件发生的表的 ID

+   `reporting`

    其中之一是`updated`、`all`、`subscribe`或`DDL`

+   `columns`

    受事件影响的列的逗号分隔列表

+   `table_event`

    一个或多个`INSERT`、`DELETE`、`UPDATE`、`SCAN`、`DROP`、`ALTER`、`CREATE`、`GCP_COMPLETE`、`CLUSTER_FAILURE`、`STOP`、`NODE_FAILURE`、`SUBSCRIBE`、`UNSUBSCRIBE`和`ALL`（由 NDB API 中的`Event::TableEvent`定义）

`events`表是在 NDB 8.0.29 中添加的。
