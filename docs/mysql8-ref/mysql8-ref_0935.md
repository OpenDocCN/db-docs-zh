# 15.1.37 截断表语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/truncate-table.html`](https://dev.mysql.com/doc/refman/8.0/en/truncate-table.html)

```sql
TRUNCATE [TABLE] *tbl_name*
```

`TRUNCATE TABLE`会完全清空表。它需要`DROP`权限。从逻辑上讲，`TRUNCATE TABLE`类似于删除所有行的`DELETE`语句，或一系列`DROP TABLE`和`CREATE TABLE`语句。

为了实现高性能，`TRUNCATE TABLE`绕过了删除数据的 DML 方法。因此，它不会触发`ON DELETE`触发器，不能用于具有父子外键关系的`InnoDB`表，也不能像 DML 操作那样回滚。但是，对于使用原子 DDL 支持的存储引擎的表的`TRUNCATE TABLE`操作，如果服务器在操作期间停止，则要么完全提交，要么回滚。有关更多信息，请参见第 15.1.1 节，“原子数据定义语句支持”。

虽然`TRUNCATE TABLE`类似于`DELETE`，但它被归类为 DDL 语句而不是 DML 语句。它与`DELETE`在以下方面有所不同：

+   截断操作会删除并重新创建表，这比逐行删除行要快得多，特别是对于大表而言。

+   截断操作会导致隐式提交，因此无法回滚。参见第 15.3.3 节，“导致隐式提交的语句”。

+   如果会话持有活动表锁，则无法执行截断操作。

+   如果有其他表引用该表的`FOREIGN KEY`约束，则`TRUNCATE TABLE`对`InnoDB`表或`NDB`表会失败。允许同一表的列之间的外键约束。

+   截断操作不会返回有意义的已删除行数值。通常的结果是“0 行受影响”，应该解释为“没有信息”。

+   只要表定义有效，即使数据或索引文件已损坏，也可以使用`TRUNCATE TABLE`将表重新创建为空表。

+   任何`AUTO_INCREMENT`值都将重置为其起始值。即使对于通常不重用序列值的`MyISAM`和`InnoDB`也是如此。

+   当与分区表一起使用时，`TRUNCATE TABLE`保留分区；也就是说，数据和索引文件被删除并重新创建，而分区定义不受影响。

+   `TRUNCATE TABLE`语句不会触发`ON DELETE`触发器。

+   支持截断损坏的`InnoDB`表。

对于二进制日志记录和复制的目的，`TRUNCATE TABLE`被视为 DDL 而不是 DML，并始终作为一个语句记录。

对于一个表，`TRUNCATE TABLE`会关闭所有使用`HANDLER OPEN`打开的处理程序。

在 MySQL 5.7 及更早版本中，在具有大缓冲池和启用`innodb_adaptive_hash_index`的系统上，`TRUNCATE TABLE`操作可能会导致系统性能暂时下降，因为在删除表的自适应哈希索引条目时会发生 LRU 扫描（Bug #68184）。在 MySQL 8.0 中，将`TRUNCATE TABLE`重新映射为`DROP TABLE`和`CREATE TABLE`避免了问题的 LRU 扫描。

`TRUNCATE TABLE`可以用于性能模式摘要表，但效果是将摘要列重置为 0 或`NULL`，而不是删除行。参见 Section 29.12.20, “性能模式摘要表”。

截断位于文件表表空间中的`InnoDB`表会删除现有的表空间并创建一个新的。从 MySQL 8.0.21 开始，如果表空间是在早期版本中创建的并位于未知目录中，`InnoDB`会在默认位置创建新的表空间，并将以下警告写入错误日志：数据目录位置必须在已知目录中。数据目录位置将被忽略，并且文件将被放入默认 datadir 位置。已知目录是由`datadir`、`innodb_data_home_dir`和`innodb_directories`变量定义的目录。要让`TRUNCATE TABLE`在当前位置创建表空间，请在运行`TRUNCATE TABLE`之前将目录添加到`innodb_directories`设置中。
