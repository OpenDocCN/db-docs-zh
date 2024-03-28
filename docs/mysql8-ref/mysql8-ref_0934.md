# 15.1.36 RENAME TABLE Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/rename-table.html`](https://dev.mysql.com/doc/refman/8.0/en/rename-table.html)

```sql
RENAME TABLE
    *tbl_name* TO *new_tbl_name*
    [, *tbl_name2* TO *new_tbl_name2*] ...
```

`RENAME TABLE`重命名一个或多个表。你必须对原始表具有`ALTER`和`DROP`权限，对新表具有`CREATE`和`INSERT`权限。

例如，要将名为`old_table`的表重命名为`new_table`，可以使用以下语句：

```sql
RENAME TABLE old_table TO new_table;
```

该语句等同于以下`ALTER TABLE`语句：

```sql
ALTER TABLE old_table RENAME new_table;
```

`RENAME TABLE`，与`ALTER TABLE`不同，可以在单个语句中重命名多个表：

```sql
RENAME TABLE old_table1 TO new_table1,
             old_table2 TO new_table2,
             old_table3 TO new_table3;
```

重命名操作是从左到右执行的。因此，要交换两个表名，可以这样做（假设中间名为`tmp_table`的表不存在）：

```sql
RENAME TABLE old_table TO tmp_table,
             new_table TO old_table,
             tmp_table TO new_table;
```

表的元数据锁按名称顺序获取，在某些情况下，当多个事务同时执行时，这可能会影响操作结果。参见 Section 10.11.4, “Metadata Locking”。

从 MySQL 8.0.13 开始，可以重命名被`LOCK TABLES`语句锁定的表，前提是它们被使用`WRITE`锁定或是在多表重命名操作中从之前步骤中重命名`WRITE`锁定的表的结果。例如，这是允许的：

```sql
LOCK TABLE old_table1 WRITE;
RENAME TABLE old_table1 TO new_table1,
             new_table1 TO new_table2;
```

这是不允许的：

```sql
LOCK TABLE old_table1 READ;
RENAME TABLE old_table1 TO new_table1,
             new_table1 TO new_table2;
```

在 MySQL 8.0.13 之前，要执行`RENAME TABLE`，不能有使用`LOCK TABLES`锁定的表。

满足事务表锁定条件后，重命名操作是原子性的；在重命名过程中，其他会话无法访问任何表。

如果在`RENAME TABLE`过程中发生任何错误，该语句将失败，不会进行任何更改。

你可以使用`RENAME TABLE`将一个表从一个数据库移动到另一个数据库：

```sql
RENAME TABLE *current_db.tbl_name* TO *other_db.tbl_name;*
```

使用这种方法将所有表从一个数据库移动到另一个数据库实际上是重命名数据库（MySQL 没有单个语句执行此操作），只是原始数据库仍然存在，尽管没有表。

像`RENAME TABLE`一样，`ALTER TABLE ... RENAME`也可以用于将表移动到不同的数据库。无论使用哪种语句，如果重命名操作将表移动到位于不同文件系统上的数据库，操作结果的成功与否取决于特定平台，并取决于用于移动表文件的底层操作系统调用。

如果一个表有触发器，尝试将表重命名到不同的数据库会失败，并显示错误 Trigger in wrong schema (`ER_TRG_IN_WRONG_SCHEMA`)。

未加密的表可以移动到启用加密的数据库，反之亦然。但是，如果启用了`table_encryption_privilege_check`变量，则如果表的加密设置与默认数据库加密不同，则需要`TABLE_ENCRYPTION_ADMIN`权限。

要重命名`TEMPORARY`表，`RENAME TABLE` 不起作用。请改用`ALTER TABLE`。

`RENAME TABLE` 对视图有效，但视图不能重命名到不同的数据库。

为重命名的表或视图专门授予的任何权限不会迁移到新名称。它们必须手动更改。

`RENAME TABLE *`tbl_name`* TO *`new_tbl_name`*` 会更改内部生成的以及以字符串“*`tbl_name`*_ibfk_”开头的用户定义的外键约束名称，以反映新表名。`InnoDB`将以字符串“*`tbl_name`*_ibfk_”开头的外键约束名称解释为内部生成的名称。

指向重命名表的外键约束名称会自动更新，除非存在冲突，否则语句将因错误而失败。如果重命名的约束名称已经存在，则会发生冲突。在这种情况下，您必须删除并重新创建外键以使其正常工作。

`RENAME TABLE *`tbl_name`* TO *`new_tbl_name`*` 会更改以字符串“*`tbl_name`*_chk_”开头的内部生成和用户定义的`CHECK`约束名称，以反映新表名。MySQL 将以字符串“*`tbl_name`*_chk_”开头的`CHECK`约束名称解释为内部生成的名称。示例：

```sql
mysql> SHOW CREATE TABLE t1\G
*************************** 1\. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `i1` int(11) DEFAULT NULL,
  `i2` int(11) DEFAULT NULL,
  CONSTRAINT `t1_chk_1` CHECK ((`i1` > 0)),
  CONSTRAINT `t1_chk_2` CHECK ((`i2` < 0))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.02 sec)

mysql> RENAME TABLE t1 TO t3;
Query OK, 0 rows affected (0.03 sec)

mysql> SHOW CREATE TABLE t3\G
*************************** 1\. row ***************************
       Table: t3
Create Table: CREATE TABLE `t3` (
  `i1` int(11) DEFAULT NULL,
  `i2` int(11) DEFAULT NULL,
  CONSTRAINT `t3_chk_1` CHECK ((`i1` > 0)),
  CONSTRAINT `t3_chk_2` CHECK ((`i2` < 0))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.01 sec)
```
