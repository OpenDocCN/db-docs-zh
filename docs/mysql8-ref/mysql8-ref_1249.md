# 17.15.1 关于压缩的 InnoDB INFORMATION_SCHEMA 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-compression-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-compression-tables.html)

17.15.1.1 INNODB_CMP 和 INNODB_CMP_RESET

17.15.1.2 INNODB_CMPMEM 和 INNODB_CMPMEM_RESET

17.15.1.3 使用压缩信息模式表

有两对`InnoDB` `INFORMATION_SCHEMA`关于压缩的表，可以提供关于整体压缩效果的见解：

+   `INNODB_CMP` 和 `INNODB_CMP_RESET` 提供有关压缩操作次数和执行压缩所花费时间的信息。

+   `INNODB_CMPMEM` 和 `INNODB_CMPMEM_RESET` 提供有关为压缩分配内存的信息。
