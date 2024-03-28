# 10.5.7 优化 InnoDB DDL 操作

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-ddl-operations.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-ddl-operations.html)

+   表和索引的许多 DDL 操作（`CREATE`，`ALTER`和`DROP`语句）可以在线执行。详细信息请参见第 17.12 节，“InnoDB 和在线 DDL”。

+   支持在线 DDL 添加辅助索引意味着通常可以通过在加载数据后添加辅助索引来加快创建和加载表及相关索引的过程。

+   使用`TRUNCATE TABLE`来清空表，而不是`DELETE FROM *`tbl_name`*`。外键约束可能会使`TRUNCATE`语句像常规的`DELETE`语句一样工作，在这种情况下，一系列命令，如`DROP TABLE`和`CREATE TABLE`可能是最快的。

+   因为主键对于每个`InnoDB`表的存储布局至关重要，并且更改主键的定义涉及重新组织整个表，因此始终将主键设置为`CREATE TABLE`语句的一部分，并提前计划，以便之后不需要`ALTER`或`DROP`主键。
