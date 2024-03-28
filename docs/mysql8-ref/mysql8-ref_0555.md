# 10.3.4 外键优化

> 原文：[`dev.mysql.com/doc/refman/8.0/en/foreign-key-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/foreign-key-optimization.html)

如果一张表有很多列，并且您查询许多不同的列组合，将不经常使用的数据拆分到每个只有几列的单独表中，然后通过从主表复制数值 ID 列将它们与主表关联起来可能是有效的。这样，每个小表都可以有一个用于快速查找其数据的主键，并且您可以使用连接操作仅查询您需要的列集。根据数据的分布方式，查询可能执行更少的 I/O 并占用更少的缓存内存，因为相关列在磁盘上紧密打包在一起。（为了最大化性能，查询尝试从磁盘读取尽可能少的数据块；只有几列的表可以在每个数据块中容纳更多行。）