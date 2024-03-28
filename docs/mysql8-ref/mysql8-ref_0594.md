# 10.7 为 MEMORY 表进行优化

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-memory-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-memory-tables.html)

考虑使用`MEMORY`表存储经常访问的非关键数据，这些数据是只读的或很少更新的。通过在真实工作负载下将应用程序与等效的`InnoDB`或`MyISAM`表进行基准测试，以确认任何额外性能是否值得冒失去数据的风险，或者在应用程序启动时从基于磁盘的表复制数据的开销。

为了获得`MEMORY`表的最佳性能，请检查针对每个表的查询类型，并为每个关联索引指定要使用的类型，可以是 B-tree 索引或哈希索引。在`CREATE INDEX`语句中，使用`USING BTREE`或`USING HASH`子句。B-tree 索引对于通过诸如`>`或`BETWEEN`等运算符进行大于或小于比较的查询非常快速。哈希索引仅对通过`=`运算符查找单个值或通过`IN`运算符查找一组受限制的值的查询非常快速。关于为什么`USING BTREE`通常比默认的`USING HASH`更好的选择，请参见 Section 10.2.1.23, “Avoiding Full Table Scans”。有关不同类型`MEMORY`索引的实现细节，请参见 Section 10.3.9, “Comparison of B-Tree and Hash Indexes”。
