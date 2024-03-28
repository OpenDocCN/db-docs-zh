# 17.23 InnoDB 限制和限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-restrictions-limitations.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-restrictions-limitations.html)

本节描述了 `InnoDB` 存储引擎的限制和限制。

+   不能创建具有与内部 `InnoDB` 列名匹配的列名的表（包括 `DB_ROW_ID`、`DB_TRX_ID` 和 `DB_ROLL_PTR`）。此限制适用于任何大小写形式中的名称使用。

    ```sql
    mysql> CREATE TABLE t1 (c1 INT, db_row_id INT) ENGINE=INNODB;
    ERROR 1166 (42000): Incorrect column name 'db_row_id'
    ```

+   `SHOW TABLE STATUS` 对于 `InnoDB` 表不提供准确的统计信息，除了表保留的物理大小。行计数仅是 SQL 优化中使用的粗略估计。

+   `InnoDB` 不会保留表中行的内部计数，因为并发事务可能在同一时间“看到”不同数量的行。因此，`SELECT COUNT(*)` 语句仅计算当前事务可见的行数。

    有关 `InnoDB` 如何处理 `SELECT COUNT(*)` 语句的信息，请参阅 第 14.19.1 节，“聚合函数描述” 中的 `COUNT()` 描述。

+   `ROW_FORMAT=COMPRESSED` 不支持大于 16KB 的页面大小。

+   使用特定 `InnoDB` 页面大小（`innodb_page_size`）的 MySQL 实例不能使用来自使用不同页面大小的实例的数据文件或日志文件。

+   有关使用 *可传输表空间* 功能导入表的限制，请参见 表导入限制。

+   有关在线 DDL 的限制，请参见 第 17.12.8 节，“在线 DDL 限制”。

+   有关一般表空间的限制，请参见 一般表空间限制。

+   有关数据静态加密的限制，请参见 加密限制。
