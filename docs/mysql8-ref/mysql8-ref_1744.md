# 25.6.12 NDB 集群中的在线 ALTER TABLE 操作

> 译文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-operations.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-operations.html)

MySQL NDB 集群 8.0 支持使用 `ALTER TABLE ... ALGORITHM=DEFAULT|INPLACE|COPY` 进行在线表模式更改。NDB 集群处理 `COPY` 和 `INPLACE` 如下几段描述的方式。

对于 `ALGORITHM=COPY`，**mysqld** NDB 集群处理程序执行以下操作：

+   告诉数据节点创建表的空副本，并对此副本进行所需的模式更改。

+   从原始表中读取行，并将其写入副本。

+   告诉数据节点删除原始表，然后重命名副本。

我们有时将其称为“复制”或“离线” `ALTER TABLE`。

DML 操作不允许与复制的 `ALTER TABLE` 并发进行。

发出复制 `ALTER TABLE` 语句的 **mysqld** 获取元数据锁，但这仅在该 **mysqld** 上有效。其他 `NDB` 客户端可以在复制 `ALTER TABLE` 过程中修改行数据，导致不一致。

对于 `ALGORITHM=INPLACE`，NDB 集群处理程序告诉数据节点进行所需的更改，并且不执行任何数据复制。

我们还将其称为“非复制”或“在线” `ALTER TABLE`。

非复制 `ALTER TABLE` 允许并发的 DML 操作。

`ALGORITHM=INSTANT` 不受 NDB 8.0 支持。

无论使用的算法如何，**mysqld** 在执行 **ALTER TABLE** 时会获取全局模式锁（GSL）；这会阻止在集群中的此节点或任何其他 SQL 节点上同时执行任何（其他）DDL 或备份。通常情况下，这不会有问题，除非 `ALTER TABLE` 需要很长时间。

注意

一些较旧的 NDB 集群版本使用特定于 `NDB` 的语法进行在线 `ALTER TABLE` 操作。该语法已被移除。

在 `NDB` 表的可变宽度列上添加和删除索引的操作是在线的。在线操作是非复制的；也就是说，它们不需要重新创建索引。它们不会锁定正在被其他 API 节点访问的 NDB Cluster 中的表（但请参阅限制 NDB 在线操作，本节后面）。这些操作不需要单用户模式用于在具有多个 API 节点的 NDB 集群中进行的 `NDB` 表更改；事务可以在在线 DDL 操作期间继续无间断。

`ALGORITHM=INPLACE` 可用于在 `NDB` 表上执行在线 `ADD COLUMN`、`ADD INDEX`（包括 `CREATE INDEX` 语句）和 `DROP INDEX` 操作。还支持对 `NDB` 表进行在线重命名（在 NDB 8.0 之前，此类列无法在线重命名）。

无法在线向 `NDB` 表中添加基于磁盘的列。这意味着，如果您希望向使用表级 `STORAGE DISK` 选项的 `NDB` 表添加内存列，必须明确声明新列使用基于内存的存储。例如——假设您已经创建了表空间 `ts1`——假设您创建表 `t1` 如下：

```sql
mysql> CREATE TABLE t1 (
     >     c1 INT NOT NULL PRIMARY KEY,
     >     c2 VARCHAR(30)
     >     )
     >     TABLESPACE ts1 STORAGE DISK
     >     ENGINE NDB;
Query OK, 0 rows affected (1.73 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

您可以在线向此表添加一个新的内存列，如下所示：

```sql
mysql> ALTER TABLE t1
     >     ADD COLUMN c3 INT COLUMN_FORMAT DYNAMIC STORAGE MEMORY,
     >     ALGORITHM=INPLACE;
Query OK, 0 rows affected (1.25 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

如果省略了 `STORAGE MEMORY` 选项，则此语句将失败：

```sql
mysql> ALTER TABLE t1
     >     ADD COLUMN c4 INT COLUMN_FORMAT DYNAMIC,
     >     ALGORITHM=INPLACE;
ERROR 1846 (0A000): ALGORITHM=INPLACE is not supported. Reason:
Adding column(s) or add/reorganize partition not supported online. Try
ALGORITHM=COPY.
```

如果省略 `COLUMN_FORMAT DYNAMIC` 选项，则会自动使用动态列格式，但会发出警告，如下所示：

```sql
mysql> ALTER ONLINE TABLE t1 ADD COLUMN c4 INT STORAGE MEMORY;
Query OK, 0 rows affected, 1 warning (1.17 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Warning
   Code: 1478
Message: DYNAMIC column c4 with STORAGE DISK is not supported, column will
become FIXED 
mysql> SHOW CREATE TABLE t1\G
*************************** 1\. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `c1` int(11) NOT NULL,
  `c2` varchar(30) DEFAULT NULL,
  `c3` int(11) /*!50606 STORAGE MEMORY */ /*!50606 COLUMN_FORMAT DYNAMIC */ DEFAULT NULL,
  `c4` int(11) /*!50606 STORAGE MEMORY */ DEFAULT NULL,
  PRIMARY KEY (`c1`)
) /*!50606 TABLESPACE ts_1 STORAGE DISK */ ENGINE=ndbcluster DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.03 sec)
```

注意

`STORAGE` 和 `COLUMN_FORMAT` 关键字仅在 NDB Cluster 中受支持；在 MySQL 的任何其他版本中，尝试在 `CREATE TABLE` 或 `ALTER TABLE` 语句中使用这两个关键字会导致错误。

也可以在 `NDB` 表上使用语句 `ALTER TABLE ... REORGANIZE PARTITION, ALGORITHM=INPLACE`，在没有 `*`partition_names`* INTO (*`partition_definitions`*)` 选项的情况下。这可以用于在线在新添加到集群中的数据节点之间重新分配 NDB Cluster 数据。这不会执行任何碎片整理，这需要一个 `OPTIMIZE TABLE` 或空 `ALTER TABLE` 语句。更多信息，请参见 Section 25.6.7, “Adding NDB Cluster Data Nodes Online”。

#### NDB 在线操作的限制

不支持在线`DROP COLUMN`操作。

添加列或添加或删除索引的在线`ALTER TABLE`、`CREATE INDEX`或`DROP INDEX`语句受以下限制：

+   给定的在线`ALTER TABLE`只能使用`ADD COLUMN`、`ADD INDEX`或`DROP INDEX`中的一个。可以在单个语句中在线添加一个或多个列；在单个语句中只能创建或删除一个索引。

+   在运行在线`ALTER TABLE` `ADD COLUMN`、`ADD INDEX`或`DROP INDEX`（或`CREATE INDEX`或`DROP INDEX`语句）时，正在更改的表对于除运行在线操作的 API 节点之外的其他 API 节点不会被锁定。然而，在执行在线操作时，该表会针对*相同*API 节点上发起的任何其他操作被锁定。

+   要更改的表必须具有显式主键；由`NDB`存储引擎创建的隐藏主键不足以满足此目的。

+   表使用的存储引擎无法在线更改。

+   表使用的表空间无法在线更改。从 NDB 8.0.21 开始，类似`ALTER TABLE *`ndb_table`* ... ALGORITHM=INPLACE, TABLESPACE=*`new_tablespace`*`的语句被明确禁止。（Bug #99269，Bug #31180526）

+   在与 NDB Cluster Disk Data 表一起使用时，无法在线更改列的存储类型（`DISK`或`MEMORY`）。这意味着，当以在线方式添加或删除索引时，如果要更改列或列的存储类型，必须在添加或删除索引的语句中使用`ALGORITHM=COPY`。

要在线添加的列不能使用`BLOB`或`TEXT`类型，并且必须满足以下条件：

+   列必须是动态的；也就是说，必须能够使用`COLUMN_FORMAT DYNAMIC`来创建它们。如果省略`COLUMN_FORMAT DYNAMIC`选项，则动态列格式会自动使用。

+   列必须允许`NULL`值，并且除`NULL`之外不能有任何显式默认值。在线添加的列会自动创建为`DEFAULT NULL`，如下所示：

    ```sql
    mysql> CREATE TABLE t2 (
         >     c1 INT NOT NULL AUTO_INCREMENT PRIMARY KEY
         >     ) ENGINE=NDB;
    Query OK, 0 rows affected (1.44 sec)

    mysql> ALTER TABLE t2
         >     ADD COLUMN c2 INT,
         >     ADD COLUMN c3 INT,
         >     ALGORITHM=INPLACE;
    Query OK, 0 rows affected, 2 warnings (0.93 sec)

    mysql> SHOW CREATE TABLE t1\G
    *************************** 1\. row ***************************
           Table: t1
    Create Table: CREATE TABLE `t2` (
      `c1` int(11) NOT NULL AUTO_INCREMENT,
      `c2` int(11) DEFAULT NULL,
      `c3` int(11) DEFAULT NULL,
      PRIMARY KEY (`c1`)
    ) ENGINE=ndbcluster DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.00 sec)
    ```

+   必须在任何现有列之后添加列。如果尝试在线在任何现有列之前或使用`FIRST`关键字添加列，则语句将失败并显示错误。

+   无法在线重新排序现有表列。

对于`NDB`表的在线`ALTER TABLE`操作，在线添加列时，或者在线创建或删除索引时，固定格式列会被转换为动态格式，如下所示（为了清晰起见，重复显示刚刚显示的`CREATE TABLE`和`ALTER TABLE`语句）：

```sql
mysql> CREATE TABLE t2 (
     >     c1 INT NOT NULL AUTO_INCREMENT PRIMARY KEY
     >     ) ENGINE=NDB;
Query OK, 0 rows affected (1.44 sec)

mysql> ALTER TABLE t2
     >     ADD COLUMN c2 INT,
     >     ADD COLUMN c3 INT,
     >     ALGORITHM=INPLACE;
Query OK, 0 rows affected, 2 warnings (0.93 sec)

mysql> SHOW WARNINGS;
*************************** 1\. row ***************************
  Level: Warning
   Code: 1478
Message: Converted FIXED field 'c2' to DYNAMIC to enable online ADD COLUMN
*************************** 2\. row ***************************
  Level: Warning
   Code: 1478
Message: Converted FIXED field 'c3' to DYNAMIC to enable online ADD COLUMN 2 rows in set (0.00 sec)
```

只有在线添加的列或列必须是动态的。现有列不需要；这包括表的主键，也可以是`FIXED`，如下所示：

```sql
mysql> CREATE TABLE t3 (
     >     c1 INT NOT NULL AUTO_INCREMENT PRIMARY KEY COLUMN_FORMAT FIXED
     >     ) ENGINE=NDB;
Query OK, 0 rows affected (2.10 sec)

mysql> ALTER TABLE t3 ADD COLUMN c2 INT, ALGORITHM=INPLACE;
Query OK, 0 rows affected, 1 warning (0.78 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW WARNINGS;
*************************** 1\. row ***************************
  Level: Warning
   Code: 1478
Message: Converted FIXED field 'c2' to DYNAMIC to enable online ADD COLUMN 1 row in set (0.00 sec)
```

通过重命名操作不会将列从`FIXED`转换为`DYNAMIC`列格式。有关`COLUMN_FORMAT`的更多信息，请参见第 15.1.20 节，“CREATE TABLE Statement”。

在使用`ALGORITHM=INPLACE`的`ALTER TABLE`语句中支持`KEY`、`CONSTRAINT`和`IGNORE`关键字。

使用在线`ALTER TABLE`语句将`MAX_ROWS`设置为 0 是不允许的。您必须使用复制的`ALTER TABLE`来执行此操作。（Bug #21960004）
