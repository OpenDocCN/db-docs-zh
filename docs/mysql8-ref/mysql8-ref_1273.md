# 17.19 InnoDB 和 MySQL 复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-and-mysql-replication.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-and-mysql-replication.html)

可以以副本上的存储引擎与源端不同的方式使用复制。例如，您可以将对源端上的`InnoDB`表的修改复制到副本上的`MyISAM`表。有关更多信息，请参见第 19.4.4 节，“使用不同源和副本存储引擎的复制”。

有关设置副本的信息，请参见第 19.1.2.6 节，“设置副本”和第 19.1.2.5 节，“选择数据快照方法”。要创建一个新的副本而不关闭源端或现有副本，请使用 MySQL 企业备份产品。

在源端失败的事务不会影响复制。MySQL 复制基于二进制日志，MySQL 在其中写入修改数据的 SQL 语句。失败的事务（例如，由于外键违反或回滚而失败）不会被写入二进制日志，因此不会被发送到副本。参见第 15.3.1 节，“START TRANSACTION，COMMIT 和 ROLLBACK 语句”。

**复制和级联。** 仅当源端和副本上共享外键关系的表都使用`InnoDB`时，源端上`InnoDB`表的级联操作才会在副本上执行。无论您使用基于语句还是基于行的复制，这都是正确的。假设您已经开始了复制，然后在源端创建两个表，其中`InnoDB`被定义为默认存储引擎，使用以下`CREATE TABLE`语句：

```sql
CREATE TABLE fc1 (
    i INT PRIMARY KEY,
    j INT
);

CREATE TABLE fc2 (
    m INT PRIMARY KEY,
    n INT,
    FOREIGN KEY ni (n) REFERENCES fc1 (i)
        ON DELETE CASCADE
);
```

如果副本将`MyISAM`定义为默认存储引擎，则在副本上创建相同的表，但它们使用`MyISAM`存储引擎，并且`FOREIGN KEY`选项被忽略。现在我们在源端的表中插入一些行：

```sql
source> INSERT INTO fc1 VALUES (1, 1), (2, 2);
Query OK, 2 rows affected (0.09 sec)
Records: 2  Duplicates: 0  Warnings: 0

source> INSERT INTO fc2 VALUES (1, 1), (2, 2), (3, 1);
Query OK, 3 rows affected (0.19 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

此时，在源端和副本上，`fc1`表包含 2 行，`fc2`表包含 3 行，如下所示：

```sql
source> SELECT * FROM fc1;
+---+------+
| i | j    |
+---+------+
| 1 |    1 |
| 2 |    2 |
+---+------+
2 rows in set (0.00 sec)

source> SELECT * FROM fc2;
+---+------+
| m | n    |
+---+------+
| 1 |    1 |
| 2 |    2 |
| 3 |    1 |
+---+------+
3 rows in set (0.00 sec)

replica> SELECT * FROM fc1;
+---+------+
| i | j    |
+---+------+
| 1 |    1 |
| 2 |    2 |
+---+------+
2 rows in set (0.00 sec)

replica> SELECT * FROM fc2;
+---+------+
| m | n    |
+---+------+
| 1 |    1 |
| 2 |    2 |
| 3 |    1 |
+---+------+
3 rows in set (0.00 sec)
```

现在假设您在源端执行以下`DELETE`语句：

```sql
source> DELETE FROM fc1 WHERE i=1;
Query OK, 1 row affected (0.09 sec)
```

由于级联，源端的`fc2`表现在只包含 1 行：

```sql
source> SELECT * FROM fc2;
+---+---+
| m | n |
+---+---+
| 2 | 2 |
+---+---+
1 row in set (0.00 sec)
```

然而，在副本上级联不会传播，因为在副本上对`fc1`的`DELETE`不会从`fc2`中删除任何行。副本中的`fc2`的副本仍然包含最初插入的所有行：

```sql
replica> SELECT * FROM fc2;
+---+---+
| m | n |
+---+---+
| 1 | 1 |
| 3 | 1 |
| 2 | 2 |
+---+---+
3 rows in set (0.00 sec)
```

这种差异是由于级联删除是由`InnoDB`存储引擎内部处理的，这意味着没有任何更改被记录。
