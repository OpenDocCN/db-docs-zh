# 17.12.4 在线 DDL 内存管理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/online-ddl-memory-management.html`](https://dev.mysql.com/doc/refman/8.0/en/online-ddl-memory-management.html)

在线 DDL 操作在创建或重建二级索引的不同阶段分配临时缓冲区。`innodb_ddl_buffer_size` 变量，于 MySQL 8.0.27 中引入，定义了在线 DDL 操作的最大缓冲区大小。默认设置为 1048576 字节（1 MB）。该设置适用于执行在线 DDL 操作的线程创建的缓冲区。定义适当的缓冲区大小限制可避免在线 DDL 操作创建或重建二级索引时出现潜在的内存不足错误。每个 DDL 线程的最大缓冲区大小是最大缓冲区大小除以 DDL 线程数（`innodb_ddl_buffer_size`/`innodb_ddl_threads`）。

在 MySQL 8.0.27 之前，`innodb_sort_buffer_size` 变量定义了在线 DDL 操作创建或重建二级索引的缓冲区大小。
