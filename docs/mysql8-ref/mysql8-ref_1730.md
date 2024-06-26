> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-add-node-remarks.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-add-node-remarks.html)

#### 25.6.7.1 添加 NDB 集群数据节点在线：一般问题

本节提供关于在线添加 NDB 集群节点的行为和当前限制的一般信息。

**数据重新分布。** 添加新节点的在线功能包括通过`ALTER TABLE ... REORGANIZE PARTITION`语句重新组织`NDBCLUSTER`表数据和索引，使其分布在所有数据节点上，包括新节点。支持内存和磁盘数据表的表重组。此重新分布目前不包括唯一索引（只重新分布有序索引）。

在添加新数据节点之前已存在的`NDBCLUSTER`表的重新分布不是自动的，但可以通过**mysql**或其他 MySQL 客户端应用程序中的简单 SQL 语句来完成。但是，在添加新节点组后创建的表中添加的所有数据和索引会自动分布在所有集群数据节点中，包括作为新节点组的一部分添加的节点。

**部分启动。** 可以添加一个新节点组，而不是所有新数据节点都已启动。也可以将新节点组添加到降级集群中，即只部分启动的集群，或其中一个或多个数据节点未运行。在后一种情况下，必须有足够的节点运行以使集群可行，然后才能添加新节点组。

**对正在进行的操作的影响。** 使用 NDB 集群数据的正常 DML 操作不会受到新节点组的创建或添加，或表重组的影响。但是，在表重组期间无法同时执行 DDL，也就是说，在执行`ALTER TABLE ... REORGANIZE PARTITION`语句时无法发出其他 DDL 语句。此外，在执行`ALTER TABLE ... REORGANIZE PARTITION`（或执行任何其他 DDL 语句）期间，无法重新启动集群数据节点。

**故障处理。** 在节点组创建和表重组期间数据节点的故障处理如下表所示：

**表 25.66 节点组创建和表重组期间数据节点故障处理**

| 在期间发生故障 | 在“旧”数据节点中发生故障 | 在“新”数据节点中发生故障 | 系统故障 |
| --- | --- | --- | --- |
| 节点组创建 |

+   **如果非主节点失败：** 节点组的创建总是向前滚动。

+   **如果主节点失败：**

    +   **如果内部提交点已经达到：** 节点组的创建将被前滚。

    +   **如果内部提交点尚未达到。** 节点组的创建将被回滚。

|

+   **如果除主节点之外的节点失败：** 节点组的创建总是被前滚。

+   **如果主节点失败：**

    +   **如果内部提交点已经达到：** 节点组的创建将被前滚。

    +   **如果内部提交点尚未达到。** 节点组的创建将被回滚。

|

+   **如果执行 CREATE NODEGROUP 已经达到内部提交点：** 当重新启动时，集群将包括新的节点组。否则不包括。

+   **如果执行 CREATE NODEGROUP 尚未达到内部提交点：** 当重新启动时，集群不包括新的节点组。

|

| 表重新组织 |
| --- |

+   **如果除主节点之外的节点失败：** 表重新组织总是被前滚。

+   **如果主节点失败：**

    +   **如果内部提交点已经达到：** 表重新组织将被前滚。

    +   **如果内部提交点尚未达到。** 表重新组织将被回滚。

|

+   **如果除主节点之外的节点失败：** 表重新组织总是被前滚。

+   **如果主节点失败：**

    +   **如果内部提交点已经达到：** 表重新组织将被前滚。

    +   **如果内部提交点尚未达到。** 表重新组织将被回滚。

|

+   **如果执行 ALTER TABLE ... REORGANIZE PARTITION 语句已经达到内部提交点：** 当集群重新启动时，属于*`table`*的数据和索引将使用“新”数据节点进行分发。

+   **如果执行 ALTER TABLE ... REORGANIZE PARTITION 语句尚未达到内部提交点：** 当集群重新启动时，属于*`table`*的数据和索引将仅使用“旧”数据节点进行分发。

|

**删除节点组。** **ndb_mgm**客户端支持`DROP NODEGROUP`命令，但只有当节点组中的数据节点不包含任何数据时才能删除节点组。由于目前没有办法“清空”特定数据节点或节点组，此命令仅适用于以下两种情况：

1.  在**ndb_mgm**客户端中发出`CREATE NODEGROUP`命令之后，在**mysql**客户端中发出任何`ALTER TABLE ... REORGANIZE PARTITION`语句之前。

1.  在使用 `DROP TABLE` 删除所有 `NDBCLUSTER` 表之后。

    `TRUNCATE TABLE` 不适用于此目的，因为数据节点继续存储表定义。
