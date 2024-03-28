> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-mysqldb.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-mysqldb.html)

#### 19.5.1.22 mysql 系统模式的复制

对`mysql`模式中的表进行的数据修改语句根据`binlog_format`的值进行复制；如果该值为`MIXED`，则使用基于行的格式复制这些语句。然而，通常间接更新此信息的语句，如`GRANT`、`REVOKE`以及操作触发器、存储过程和视图的语句，会使用基于语句的复制方式复制到副本。
