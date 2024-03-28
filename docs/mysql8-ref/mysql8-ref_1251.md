> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-innodb_cmpmem.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-innodb_cmpmem.html)

#### 17.15.1.2 INNODB_CMPMEM 和 INNODB_CMPMEM_RESET

`INNODB_CMPMEM`和`INNODB_CMPMEM_RESET`表提供有关位于缓冲池中的压缩页面的状态信息。请参阅第 17.9 节，“InnoDB 表和页面压缩”以获取有关压缩表和缓冲池使用的更多信息。`INNODB_CMP`和`INNODB_CMP_RESET`表应提供有关压缩的更有用的统计信息。

##### 内部细节

`InnoDB` 使用一个 buddy allocator 系统来管理分配给各种大小的页面的内存，从 1KB 到 16KB。这里描述的两个表的每一行对应一个单独的页面大小。

`INNODB_CMPMEM`和`INNODB_CMPMEM_RESET`表具有相同的内容，但从`INNODB_CMPMEM_RESET`读取会重置有关重定位操作的统计信息。例如，如果每 60 分钟归档一次`INNODB_CMPMEM_RESET`的输出，它将显示每小时的统计信息。如果你从未读取过`INNODB_CMPMEM_RESET`，而是监视`INNODB_CMPMEM`的输出，它将显示自`InnoDB`启动以来的累积统计信息。

有关表定义，请参见第 28.4.7 节，“INFORMATION_SCHEMA INNODB_CMPMEM 和 INNODB_CMPMEM_RESET 表”。
