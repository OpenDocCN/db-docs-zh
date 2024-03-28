# 10.2 优化 SQL 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/statement-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/statement-optimization.html)

10.2.1 优化 SELECT 语句

10.2.2 优化子查询、派生表、视图引用和公共表达式

10.2.3 优化 INFORMATION_SCHEMA 查询

10.2.4 优化 Performance Schema 查询

10.2.5 优化数据更改语句

10.2.6 优化数据库权限

10.2.7 其他优化技巧

数据库应用程序的核心逻辑是通过 SQL 语句执行的，无论是直接通过解释器发出还是通过 API 在后台提交。本节中的调优指南有助于加快各种 MySQL 应用程序的速度。这些指南涵盖了读取和写入数据的 SQL 操作，一般 SQL 操作的后台开销，以及在特定场景中使用的操作，如数据库监控。
