> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-innodb_cmp.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-innodb_cmp.html)

#### 17.15.1.1 INNODB_CMP 和 INNODB_CMP_RESET

`INNODB_CMP` 和 `INNODB_CMP_RESET` 表提供关于与压缩表相关操作的状态信息，这些操作在第 17.9 节，“InnoDB 表和页面压缩”中有描述。`PAGE_SIZE`列报告了压缩的页面大小。

这两个表具有相同的内容，但从`INNODB_CMP_RESET`读取会重置有关压缩和解压操作的统计信息。例如，如果您每 60 分钟归档一次`INNODB_CMP_RESET`的输出，您将看到每个小时周期的统计信息。如果您监视`INNODB_CMP`的输出（确保永远不要读取`INNODB_CMP_RESET`），您将看到自 InnoDB 启动以来的累积统计信息。

有关表定义，请参见第 28.4.6 节，“INFORMATION_SCHEMA INNODB_CMP 和 INNODB_CMP_RESET 表”。
