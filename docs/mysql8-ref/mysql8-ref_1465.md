> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-truncate.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-truncate.html)

#### 19.5.1.37 复制和 TRUNCATE TABLE

`TRUNCATE TABLE`通常被视为 DML 语句，因此在二进制日志记录模式为`ROW`或`MIXED`时，预计会使用基于行的格式进行记录和复制。然而，在以`STATEMENT`或`MIXED`模式记录或复制时，当事务隔离级别为`READ COMMITTED`或`READ UNCOMMITTED`时，这会导致在使用事务性存储引擎如`InnoDB`的表时出现问题，这种情况排除了基于语句的记录。

`TRUNCATE TABLE`在记录和复制方面被视为 DDL 而不是 DML，以便可以将其记录和复制为语句。然而，对于副本上的`InnoDB`和其他事务性表的影响仍遵循第 15.1.37 节“TRUNCATE TABLE Statement”中描述的规则。 (Bug #36763)
