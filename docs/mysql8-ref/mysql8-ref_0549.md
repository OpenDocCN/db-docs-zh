# 10.2.6 优化数据库权限

> 原文：[`dev.mysql.com/doc/refman/8.0/en/permission-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/permission-optimization.html)

权限设置越复杂，所有 SQL 语句的开销就越大。简化由`GRANT`语句建立的权限可以使 MySQL 在客户端执行语句时减少权限检查开销。例如，如果您没有授予任何表级或列级权限，服务器就不需要检查`tables_priv`和`columns_priv`表的内容。同样，如果您没有对任何账户设置资源限制，服务器就不需要执行资源计数。如果您有非常高的语句处理负载，请考虑使用简化的授权结构来减少权限检查开销。
