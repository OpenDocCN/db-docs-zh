# 15.3.3 导致隐式提交的语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/implicit-commit.html`](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html)

本节列出的语句（以及它们的任何同义词）会隐式结束当前会话中的任何活动事务，就好像在执行该语句之前已经执行了一个`COMMIT`。

大多数这些语句在执行后也会导致隐式提交。其目的是在其自己的特殊事务中处理每个这样的语句。事务控制和锁定语句是例外情况：如果在执行之前发生隐式提交，则在之后不会再发生另一个提交。

+   **数据定义语言（DDL）语句用于定义或修改数据库对象。** `ALTER EVENT`, `ALTER FUNCTION`, `ALTER PROCEDURE`, `ALTER SERVER`, `ALTER TABLE`, `ALTER TABLESPACE`, `ALTER VIEW`, `CREATE DATABASE`, `CREATE EVENT`, `CREATE FUNCTION`, `CREATE INDEX`, `CREATE PROCEDURE`, `CREATE ROLE`, `CREATE SERVER`, `CREATE SPATIAL REFERENCE SYSTEM`, `CREATE TABLE`, `CREATE TABLESPACE`, `CREATE TRIGGER`, `CREATE VIEW`, `DROP DATABASE`, `DROP EVENT`, `DROP FUNCTION`, `DROP INDEX`, `DROP PROCEDURE`, `DROP ROLE`, `DROP SERVER`, `DROP SPATIAL REFERENCE SYSTEM`, `DROP TABLE`, `DROP TABLESPACE`, `DROP TRIGGER`, `DROP VIEW`, `INSTALL PLUGIN`, `RENAME TABLE`, `TRUNCATE TABLE`, `UNINSTALL PLUGIN`.

    使用 `TEMPORARY` 关键字的 `CREATE TABLE` 和 `DROP TABLE` 语句不会提交事务。（这不适用于对临时表的其他操作，如 `ALTER TABLE` 和 `CREATE INDEX`，这些操作会导致提交。）然而，虽然没有隐式提交，但也不能回滚该语句，这意味着使用这些语句会违反事务的原子性。例如，如果使用 `CREATE TEMPORARY TABLE` 然后回滚事务，表仍然存在。

    在 `InnoDB` 中，`CREATE TABLE` 语句被处理为单个事务。这意味着用户的 `ROLLBACK` 不会撤消用户在该事务期间进行的 `CREATE TABLE` 语句。

    当创建非临时表时，`CREATE TABLE ... SELECT` 在执行语句之前和之后会导致隐式提交。（对于 `CREATE TEMPORARY TABLE ... SELECT` 不会发生提交。）

+   **隐式使用或修改 `mysql` 数据库中表的语句。** `ALTER USER`, `CREATE USER`, `DROP USER`, `GRANT`, `RENAME USER`, `REVOKE`, `SET PASSWORD`.

+   **事务控制和锁定语句。** `BEGIN`, `LOCK TABLES`, 如果值尚未为 1，则 `SET autocommit = 1`， `START TRANSACTION`, `UNLOCK TABLES`.

    只有当使用 `LOCK TABLES` 获取非事务表锁时，`UNLOCK TABLES` 才会提交事务。对于跟随 `FLUSH TABLES WITH READ LOCK` 的 `UNLOCK TABLES` 不会发生提交，因为后者不会获取表级锁。

    事务不能嵌套。这是在发出`START TRANSACTION`语句或其同义词时为当前事务执行的隐式提交的结果。

    导致隐式提交的语句不能在事务处于`ACTIVE`状态时用于 XA 事务。

    `BEGIN`语句与开始`BEGIN ... END`复合语句的`BEGIN`关键字的使用不同。后者不会导致隐式提交。请参阅 Section 15.6.1, “BEGIN ... END Compound Statement”。

+   **数据加载语句。** `LOAD DATA`。`LOAD DATA`仅对使用`NDB`存储引擎的表造成隐式提交。

+   **管理语句。** `ANALYZE TABLE`，`CACHE INDEX`，`CHECK TABLE`，`FLUSH`，`LOAD INDEX INTO CACHE`，`OPTIMIZE TABLE`，`REPAIR TABLE`，`RESET`（但不包括`RESET PERSIST`）。

+   **复制控制语句**。`START REPLICA`，`STOP REPLICA`，`RESET REPLICA`，`CHANGE REPLICATION SOURCE TO`，`CHANGE MASTER TO`。在 MySQL 8.0.22 中，SLAVE 关键字被 REPLICA 替换。
