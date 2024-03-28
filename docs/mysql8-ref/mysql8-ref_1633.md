> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-performance.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-performance.html)

#### 25.2.7.7 关于 NDB Cluster 性能的限制

以下性能问题特定于 NDB Cluster 或在 NDB Cluster 中特别明显：

+   **范围扫描。** 由于对`NDB`存储引擎的顺序访问，存在查询性能问题；相对于`MyISAM`或`InnoDB`，进行许多范围扫描的成本也更高。

+   **范围内记录的可靠性。** `Records in range`统计信息可用，但尚未完全测试或官方支持。这可能导致某些情况下的非最佳查询计划。如有必要，您可以使用`USE INDEX`或`FORCE INDEX`来更改执行计划。有关如何执行此操作的更多信息，请参见第 10.9.4 节，“索引提示”。

+   **唯一哈希索引。** 使用`USING HASH`创建的唯一哈希索引，如果键的一部分是`NULL`，则无法用于访问表。
