# 17.12.5 配置在线 DDL 操作的并行线程

> 原文：[`dev.mysql.com/doc/refman/8.0/en/online-ddl-parallel-thread-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/online-ddl-parallel-thread-configuration.html)

在创建或重建二级索引的在线 DDL 操作的工作流程中涉及：

+   扫描聚簇索引并将数据写入临时排序文件

+   对数据进行排序

+   从临时排序文件加载排序后的数据到二级索引中

可用于扫描聚簇索引的并行线程数由`innodb_parallel_read_threads`变量定义。默认设置为 4。最大设置为 256，这是所有会话的最大数。扫描聚簇索引的实际线程数是由`innodb_parallel_read_threads`设置或要扫描的索引子树数中较小的那个数定义的。如果达到线程限制，会话将回退到使用单个线程。

控制排序和加载数据的并行线程数由`innodb_ddl_threads`变量控制，该变量在 MySQL 8.0.27 中引入。默认设置为 4。在 MySQL 8.0.27 之前，排序和加载操作是单线程的。

以下限制适用：

+   不支持用于构建包含虚拟列的索引的并行线程。

+   完全文本索引创建不支持并行线程。

+   不支持并行线程用于空间索引创建。

+   在定义具有虚拟列的表上不支持并行扫描。

+   在定义具有全文本索引的表上不支持并行扫描。

+   在定义具有空间索引的表上不支持并行扫描。
