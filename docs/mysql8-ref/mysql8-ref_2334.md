> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-table-problems.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-table-problems.html)

#### B.3.6.1 ALTER TABLE 存在的问题

如果在使用`ALTER TABLE`更改字符列的字符集或校对规则时出现重复键错误，原因可能是新列校对规则将两个键映射到相同值，或者表已损坏。在后一种情况下，您应该对表运行`REPAIR TABLE`。`REPAIR TABLE`适用于`MyISAM`、`ARCHIVE`和`CSV`表。

如果您在事务表上使用`ALTER TABLE`，或者您正在使用 Windows 系统，`ALTER TABLE`会在您对其执行`LOCK TABLE`后解锁表。这是因为`InnoDB`和这些操作系统无法删除正在使用的表。
