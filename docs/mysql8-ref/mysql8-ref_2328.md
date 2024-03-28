> 原文：[`dev.mysql.com/doc/refman/8.0/en/nontransactional-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/nontransactional-tables.html)

#### B.3.4.5 非事务性表的回滚失败

如果在尝试执行`ROLLBACK`时收到以下消息，意味着您在事务中使用的一个或多个表不支持事务：

```sql
Warning: Some non-transactional changed tables couldn't be rolled back
```

这些非事务性表不受`ROLLBACK`语句的影响。

如果您在事务中没有故意混合事务性和非事务性表，那么出现此消息的最可能原因是您认为是事务性的表实际上并非如此。如果您尝试使用不受您的**mysqld**服务器支持的事务性存储引擎（或者已在启动选项中禁用）来创建表，就会发生这种情况。如果**mysqld**不支持某个存储引擎，它会将表创建为`MyISAM`表，这是非事务性的。

您可以通过以下任一语句检查表的存储引擎：

```sql
SHOW TABLE STATUS LIKE '*tbl_name*';
SHOW CREATE TABLE *tbl_name*;
```

请参阅 Section 15.7.7.38, “SHOW TABLE STATUS Statement”和 Section 15.7.7.10, “SHOW CREATE TABLE Statement”。

要查看您的**mysqld**服务器支持的存储引擎，请使用此语句：

```sql
SHOW ENGINES;
```

详细信息请参阅 Section 15.7.7.16, “SHOW ENGINES Statement”。
