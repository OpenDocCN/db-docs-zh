> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-autocommit-commit-rollback.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-autocommit-commit-rollback.html)

#### 17.7.2.2 自动提交、提交和回滚

在`InnoDB`中，所有用户活动都发生在一个事务内。如果启用了`autocommit`模式，每个 SQL 语句都形成一个独立的事务。默认情况下，MySQL 会在每个新连接的会话中启用`autocommit`，因此如果该语句没有返回错误，MySQL 会在每个 SQL 语句后执行提交。如果语句返回错误，则提交或回滚行为取决于错误。参见第 17.21.5 节，“InnoDB 错误处理”。

通过使用显式的`START TRANSACTION`或`BEGIN`语句开始，以及使用`COMMIT`或`ROLLBACK`语句结束，启用了`autocommit`的会话可以执行多语句事务。参见第 15.3.1 节，“START TRANSACTION, COMMIT 和 ROLLBACK 语句”。

如果在使用`SET autocommit = 0`禁用了`autocommit`模式的会话中，该会话始终保持一个事务处于打开状态。`COMMIT`或`ROLLBACK`语句结束当前事务并启动一个新事务。

如果禁用了`autocommit`的会话在没有显式提交最终事务的情况下结束，MySQL 会回滚该事务。

一些语句会隐式结束一个事务，就好像在执行该语句之前执行了`COMMIT`。详情请参见第 15.3.3 节，“导致隐式提交的语句”。

一个`COMMIT`表示当前事务中所做的更改已经永久生效，并对其他会话可见。另一方面，`ROLLBACK`语句会取消当前事务所做的所有修改。`COMMIT`和`ROLLBACK`都会释放在当前事务期间设置的所有`InnoDB`锁。

##### 使用事务对 DML 操作进行分组

默认情况下，连接到 MySQL 服务器时会启用自动提交模式，这会在执行每个 SQL 语句时自动提交。如果您有其他数据库系统的经验，在那些系统中，通常会发出一系列 DML 语句并一起提交或回滚，这种操作模式可能会让您感到陌生。

要使用多语句事务，请使用 SQL 语句`SET autocommit = 0`关闭自动提交，并在每个事务结束时使用`COMMIT`或`ROLLBACK`。要保持自动提交状态，请在每个事务开始时使用`START TRANSACTION`，并在结束时使用`COMMIT`或`ROLLBACK`。以下示例展示了两个事务。第一个被提交；第二个被回滚。

```sql
$> mysql test
```

```sql
mysql> CREATE TABLE customer (a INT, b CHAR (20), INDEX (a));
Query OK, 0 rows affected (0.00 sec)
mysql> -- Do a transaction with autocommit turned on.
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)
mysql> INSERT INTO customer VALUES (10, 'Heikki');
Query OK, 1 row affected (0.00 sec)
mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
mysql> -- Do another transaction with autocommit turned off.
mysql> SET autocommit=0;
Query OK, 0 rows affected (0.00 sec)
mysql> INSERT INTO customer VALUES (15, 'John');
Query OK, 1 row affected (0.00 sec)
mysql> INSERT INTO customer VALUES (20, 'Paul');
Query OK, 1 row affected (0.00 sec)
mysql> DELETE FROM customer WHERE b = 'Heikki';
Query OK, 1 row affected (0.00 sec)
mysql> -- Now we undo those last 2 inserts and the delete.
mysql> ROLLBACK;
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT * FROM customer;
+------+--------+
| a    | b      |
+------+--------+
|   10 | Heikki |
+------+--------+
1 row in set (0.00 sec)
mysql>
```

###### 客户端语言中的事务

在诸如 PHP、Perl DBI、JDBC、ODBC 或 MySQL 的标准 C 调用接口等 API 中，您可以像发送其他 SQL 语句（如`SELECT`或`INSERT`）一样，将事务控制语句（如`COMMIT`）作为字符串发送到 MySQL 服务器。一些 API 还提供单独的特殊事务提交和回滚函数或方法。
