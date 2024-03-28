> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-table-examples.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-table-examples.html)

#### 15.1.9.3 修改表示例

从这里创建一个名为`t1`的表：

```sql
CREATE TABLE t1 (a INTEGER, b CHAR(10));
```

将表从`t1`重命名为`t2`：

```sql
ALTER TABLE t1 RENAME t2;
```

将列`a`从`INTEGER`更改为`TINYINT NOT NULL`（保持名称不变），将列`b`从`CHAR(10)`更改为`CHAR(20)`，并将其重命名为`c`：

```sql
ALTER TABLE t2 MODIFY a TINYINT NOT NULL, CHANGE b c CHAR(20);
```

添加一个名为`d`的新的`TIMESTAMP`列：

```sql
ALTER TABLE t2 ADD d TIMESTAMP;
```

在列`d`上添加一个索引和在列`a`上添加一个`UNIQUE`索引：

```sql
ALTER TABLE t2 ADD INDEX (d), ADD UNIQUE (a);
```

要删除列`c`：

```sql
ALTER TABLE t2 DROP COLUMN c;
```

添加一个名为`c`的新的`AUTO_INCREMENT`整数列：

```sql
ALTER TABLE t2 ADD c INT UNSIGNED NOT NULL AUTO_INCREMENT,
  ADD PRIMARY KEY (c);
```

我们对`c`进行了索引（作为`PRIMARY KEY`），因为`AUTO_INCREMENT`列必须被索引，并且我们声明`c`为`NOT NULL`，因为主键列不能为`NULL`。

对于`NDB`表，也可以更改表或列使用的存储类型。例如，考虑一个如下所示创建的`NDB`表：

```sql
mysql> CREATE TABLE t1 (c1 INT) TABLESPACE ts_1 ENGINE NDB;
Query OK, 0 rows affected (1.27 sec)
```

要将此表转换为基于磁盘的存储，您可以使用以下`ALTER TABLE`语句：

```sql
mysql> ALTER TABLE t1 TABLESPACE ts_1 STORAGE DISK;
Query OK, 0 rows affected (2.99 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE t1\G
*************************** 1\. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `c1` int(11) DEFAULT NULL
) /*!50100 TABLESPACE ts_1 STORAGE DISK */
ENGINE=ndbcluster DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.01 sec)
```

在最初创建表时不需要引用表空间；但是，在`ALTER TABLE`中必须引用表空间：

```sql
mysql> CREATE TABLE t2 (c1 INT) ts_1 ENGINE NDB;
Query OK, 0 rows affected (1.00 sec)

mysql> ALTER TABLE t2 STORAGE DISK;
ERROR 1005 (HY000): Can't create table 'c.#sql-1750_3' (errno: 140)
mysql> ALTER TABLE t2 TABLESPACE ts_1 STORAGE DISK;
Query OK, 0 rows affected (3.42 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> SHOW CREATE TABLE t2\G
*************************** 1\. row ***************************
       Table: t1
Create Table: CREATE TABLE `t2` (
  `c1` int(11) DEFAULT NULL
) /*!50100 TABLESPACE ts_1 STORAGE DISK */
ENGINE=ndbcluster DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.01 sec)
```

要更改单个列的存储类型，可以使用`ALTER TABLE ... MODIFY [COLUMN]`。例如，假设您使用以下`CREATE TABLE`语句创建了一个具有两列的 NDB Cluster Disk Data 表：

```sql
mysql> CREATE TABLE t3 (c1 INT, c2 INT)
 ->     TABLESPACE ts_1 STORAGE DISK ENGINE NDB;
Query OK, 0 rows affected (1.34 sec)
```

要将列`c2`从基于磁盘的存储更改为内存存储，在 ALTER TABLE 语句中使用列定义中的 STORAGE MEMORY 子句，如下所示：

```sql
mysql> ALTER TABLE t3 MODIFY c2 INT STORAGE MEMORY;
Query OK, 0 rows affected (3.14 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

您可以通过类似的方式使用`STORAGE DISK`将内存列转换为基于磁盘的列。

列`c1`使用基于磁盘的存储，因为这是表的默认设置（由`CREATE TABLE`语句中的表级`STORAGE DISK`子句确定）。然而，列`c2`使用内存存储，如在 SHOW `CREATE TABLE`的输出中所示：

```sql
mysql> SHOW CREATE TABLE t3\G
*************************** 1\. row ***************************
       Table: t3
Create Table: CREATE TABLE `t3` (
  `c1` int(11) DEFAULT NULL,
  `c2` int(11) /*!50120 STORAGE MEMORY */ DEFAULT NULL
) /*!50100 TABLESPACE ts_1 STORAGE DISK */ ENGINE=ndbcluster DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.02 sec)
```

当您添加一个`AUTO_INCREMENT`列时，列值会自动填充为序列号。对于`MyISAM`表，您可以在`ALTER TABLE`之前执行`SET INSERT_ID=*`value`*`或使用`AUTO_INCREMENT=*`value`*`表选项来设置第一个序列号。

对于`MyISAM`表，如果不更改`AUTO_INCREMENT`列，则序列号不受影响。如果删除一个`AUTO_INCREMENT`列，然后添加另一个`AUTO_INCREMENT`列，数字将从 1 开始重新排序。

当使用复制时，在表中添加一个`AUTO_INCREMENT`列可能导致副本和源之间的行排序不同。这是因为行编号的顺序取决于用于表的特定存储引擎以及插入行的顺序。如果在源和副本上具有相同的顺序很重要，则必须在分配`AUTO_INCREMENT`编号之前对行进行排序。假设您想要向表`t1`添加一个`AUTO_INCREMENT`列，以下语句将生成一个新表`t2`，与`t1`相同但带有一个`AUTO_INCREMENT`列：

```sql
CREATE TABLE t2 (id INT AUTO_INCREMENT PRIMARY KEY)
SELECT * FROM t1 ORDER BY col1, col2;
```

这假设表`t1`具有列`col1`和`col2`。

这组语句还会生成一个新表`t2`，与`t1`相同，但添加了一个`AUTO_INCREMENT`列：

```sql
CREATE TABLE t2 LIKE t1;
ALTER TABLE t2 ADD id INT AUTO_INCREMENT PRIMARY KEY;
INSERT INTO t2 SELECT * FROM t1 ORDER BY col1, col2;
```

重要

要保证源和副本上的相同排序，必须在`ORDER BY`子句中引用`t1`的*所有*列。

无论使用何种方法创建和填充具有`AUTO_INCREMENT`列的副本，最后一步都是删除原始表，然后重命名副本：

```sql
DROP TABLE t1;
ALTER TABLE t2 RENAME t1;
```
