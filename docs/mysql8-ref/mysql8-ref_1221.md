# 17.9.1 InnoDB 表压缩

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-table-compression.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-table-compression.html)

17.9.1.1 表压缩概述

17.9.1.2 创建压缩表

17.9.1.3 调整 InnoDB 表的压缩

17.9.1.4 监控运行时的 InnoDB 表压缩

17.9.1.5 InnoDB 表压缩的工作原理

17.9.1.6 用于 OLTP 工作负载的压缩

17.9.1.7 SQL 压缩语法警告和错误

本节描述了`InnoDB`表压缩，支持`InnoDB`表位于 file_per_table 表空间或 general tablespaces 中的情况。表压缩是通过在`CREATE TABLE`或`ALTER TABLE`中使用`ROW_FORMAT=COMPRESSED`属性来启用的。
