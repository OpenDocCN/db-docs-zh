# 15.3 事务和锁定语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/sql-transactional-statements.html`](https://dev.mysql.com/doc/refman/8.0/en/sql-transactional-statements.html)

15.3.1 START TRANSACTION、COMMIT 和 ROLLBACK 语句

15.3.2 无法回滚的语句

15.3.3 导致隐式提交的语句

15.3.4 SAVEPOINT、ROLLBACK TO SAVEPOINT 和 RELEASE SAVEPOINT 语句

15.3.5 LOCK INSTANCE FOR BACKUP 和 UNLOCK INSTANCE 语句

15.3.6 LOCK TABLES 和 UNLOCK TABLES 语句

15.3.7 SET TRANSACTION 语句

15.3.8 XA 事务

MySQL 通过诸如`SET autocommit`、`START TRANSACTION`、`COMMIT` 和 `ROLLBACK`等语句支持本地事务（在给定的客户端会话中）。请参阅第 15.3.1 节，“START TRANSACTION, COMMIT, and ROLLBACK Statements”。XA 事务支持使 MySQL 能够参与分布式事务。请参阅第 15.3.8 节，“XA Transactions”。
