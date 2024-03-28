> 原文：[`dev.mysql.com/doc/refman/8.0/en/mrr-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/mrr-optimization.html)

#### 10.2.1.11 多范围读取优化

使用二级索引进行范围扫描读取行时，当表很大且未存储在存储引擎的缓存中时，可能会导致许多随机磁盘访问。通过磁盘扫描多范围读取（MRR）优化，MySQL 尝试通过首先仅扫描索引并收集相关行的键来减少范围扫描的随机磁盘访问次数。然后对键进行排序，最后根据主键的顺序从基表中检索行。磁盘扫描 MRR 的动机是减少随机磁盘访问次数，而是实现对基表数据的更顺序扫描。

多范围读取优化提供以下好处：

+   MRR 使数据行能够按顺序访问，而不是按随机顺序，基于索引元组。服务器获取满足查询条件的一组索引元组，根据数据行 ID 顺序对其进行排序，并使用排序后的元组按顺序检索数据行。这使数据访问更高效，成本更低。

+   MRR 可以批量处理对需要通过索引元组访问数据行的操作的请求，例如范围索引扫描和使用索引进行等值连接的操作。MRR 遍历一系列索引范围以获取符合条件的索引元组。随着这些结果的累积，它们被用于访问相应的数据行。在开始读取数据行之前，不需要获取所有索引元组。

MRR 优化不支持在虚拟生成列上创建的二级索引。`InnoDB` 支持在虚拟生成列上的二级索引。

以下情景说明了何时可以优化 MRR：

情景 A：MRR 可以用于 `InnoDB` 和 `MyISAM` 表进行索引范围扫描和等值连接操作。

1.  一部分索引元组被累积在缓冲区中。

1.  缓冲区中的元组按其数据行 ID 进行排序。

1.  数据行根据排序的索引元组序列进行访问。

情景 B：MRR 可以用于 `NDB` 表，用于多范围索引扫描或通过属性执行等值连接。

1.  一部分范围，可能是单键范围，在提交查询的中央节点上累积在缓冲区中。

1.  范围被发送到访问数据行的执行节点。

1.  访问的行被打包成数据包发送回中央节点。

1.  收到的带有数据行的数据包被放置在缓冲区中。

1.  数据行从缓冲区中读取。

当使用 MRR 时，`EXPLAIN` 输出中的 `Extra` 列显示 `Using MRR`。

如果不需要访问完整的表行即可生成查询结果，则`InnoDB`和`MyISAM`不使用 MRR。如果结果可以完全基于索引元组中的信息生成（通过覆盖索引）；MRR 提供不了任何好处。

两个`optimizer_switch`系统变量标志提供了使用 MRR 优化的接口。`mrr`标志控制是否启用 MRR。如果`mrr`被启用（`on`），`mrr_cost_based`标志控制优化器是否尝试在使用和不使用 MRR 之间做出基于成本的选择（`on`），或者在可能的情况下始终使用 MRR（`off`）。默认情况下，`mrr`为`on`，`mrr_cost_based`为`on`。请参阅第 10.9.2 节，“可切换的优化”。

对于 MRR，存储引擎使用`read_rnd_buffer_size`系统变量的值作为其缓冲区可以分配多少内存的指导。引擎最多使用`read_rnd_buffer_size`字节，并确定在单次传递中要处理的范围数量。