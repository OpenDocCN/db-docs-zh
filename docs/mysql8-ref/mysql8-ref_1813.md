> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-threads.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-threads.html)

#### 25.6.16.62 ndbinfo 线程表

`threads`表提供了关于在`NDB`内核中运行的线程的信息。

`threads`表包含以下列：

+   `node_id`

    线程所在节点的 ID

+   `thr_no`

    线程 ID（特定于此节点）

+   `thread_name`

    线程名称（线程类型）

+   `thread_description`

    线程（类型）描述

##### 笔记

展示了一个包括线程描述的 2 节点示例集群的样本输出：

```sql
mysql> SELECT * FROM threads;
+---------+--------+-------------+------------------------------------------------------------------+
| node_id | thr_no | thread_name | thread_description                                               |
+---------+--------+-------------+------------------------------------------------------------------+
|       5 |      0 | main        | main thread, schema and distribution handling                    |
|       5 |      1 | rep         | rep thread, asynch replication and proxy block handling          |
|       5 |      2 | ldm         | ldm thread, handling a set of data partitions                    |
|       5 |      3 | recv        | receive thread, performing receive and polling for new receives  |
|       6 |      0 | main        | main thread, schema and distribution handling                    |
|       6 |      1 | rep         | rep thread, asynch replication and proxy block handling          |
|       6 |      2 | ldm         | ldm thread, handling a set of data partitions                    |
|       6 |      3 | recv        | receive thread, performing receive and polling for new receives  |
+---------+--------+-------------+------------------------------------------------------------------+
8 rows in set (0.01 sec)
```

NDB 8.0.23 引入了在将`ThreadConfig`参数`main`或`rep`设置为 0 的同时保持另一个参数为 1 的可能性，此时线程名称为`main_rep`，描述为`主和副本线程，模式，分发，代理块和异步复制处理`。从 NDB 8.0.23 开始，还可以将`main`和`rep`都设置为 0，此时生成线程的名称在此表中显示为`main_rep_recv`，描述为`主，副本和接收线程，模式，分发，代理块和异步复制处理以及处理接收和轮询新接收`。
