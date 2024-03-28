# 26.6 关于分区的限制和限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-limitations.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-limitations.html)

26.6.1 分区键、主键和唯一键

26.6.2 与存储引擎相关的分区限制

26.6.3 与函数相关的分区限制

本节讨论了 MySQL 分区支持的当前限制和限制。

**禁止的结构。** 不允许在分区表达式中使用以下结构：

+   存储过程、存储函数、可加载函数或插件。

+   声明变量或用户变量。

有关允许在分区表达式中使用的 SQL 函数列表，请参见 第 26.6.3 节，“与函数相关的分区限制”。

**算术和逻辑运算符。** 在分区表达式中允许使用算术运算符 `+`, `-`, 和 `*`。然而，结果必须是整数值或 `NULL`（除了 `[LINEAR] KEY` 分区，如本章其他地方讨论的那样；有关更多信息，请参见 第 26.2 节，“分区类型”）。

`DIV` 运算符也受支持；不允许使用 `/` 运算符。

位运算符 `|`, `&`, `^`, `<<`, `>>`, 和 `~` 不允许在分区表达式中使用。

**服务器 SQL 模式。** 使用自定义分区的表不会保留创建时有效的 SQL 模式。如本手册其他地方所述（请参见 第 7.1.11 节，“服务器 SQL 模式”），许多 MySQL 函数和运算符的结果可能根据服务器 SQL 模式而变化。因此，在创建分区表后任何时候更改 SQL 模式可能导致这些表行为发生重大变化，并可能轻易导致数据损坏或丢失。因此，*强烈建议您在创建分区表后永远不要更改服务器 SQL 模式*。

对于服务器 SQL 模式中的一种更改使分区表无法使用的情况，请考虑以下 `CREATE TABLE` 语句，只有在 `NO_UNSIGNED_SUBTRACTION` 模式生效时才能成功执行：

```sql
mysql> SELECT @@sql_mode;
+------------+
| @@sql_mode |
+------------+
|            |
+------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE tu (c1 BIGINT UNSIGNED)
 ->   PARTITION BY RANGE(c1 - 10) (
 ->     PARTITION p0 VALUES LESS THAN (-5),
 ->     PARTITION p1 VALUES LESS THAN (0),
 ->     PARTITION p2 VALUES LESS THAN (5),
 ->     PARTITION p3 VALUES LESS THAN (10),
 ->     PARTITION p4 VALUES LESS THAN (MAXVALUE)
 -> );
ERROR 1563 (HY000): Partition constant is out of partition function domain 
mysql> SET sql_mode='NO_UNSIGNED_SUBTRACTION';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@sql_mode;
+-------------------------+
| @@sql_mode              |
+-------------------------+
| NO_UNSIGNED_SUBTRACTION |
+-------------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE tu (c1 BIGINT UNSIGNED)
 ->   PARTITION BY RANGE(c1 - 10) (
 ->     PARTITION p0 VALUES LESS THAN (-5),
 ->     PARTITION p1 VALUES LESS THAN (0),
 ->     PARTITION p2 VALUES LESS THAN (5),
 ->     PARTITION p3 VALUES LESS THAN (10),
 ->     PARTITION p4 VALUES LESS THAN (MAXVALUE)
 -> );
Query OK, 0 rows affected (0.05 sec)
```

如果在创建 `tu` 后移除 `NO_UNSIGNED_SUBTRACTION` 服务器 SQL 模式，则可能无法再访问此表：

```sql
mysql> SET sql_mode='';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM tu;
ERROR 1563 (HY000): Partition constant is out of partition function domain
mysql> INSERT INTO tu VALUES (20);
ERROR 1563 (HY000): Partition constant is out of partition function domain
```

另请参阅 第 7.1.11 节，“服务器 SQL 模式”。

服务器 SQL 模式也会影响分区表的复制。源和副本上不同的 SQL 模式可能导致分区表达式的评估不同；这可能导致数据在源和副本的给定表的副本之间分布不同，并且甚至可能导致在源上成功的分区表插入在副本上失败。为获得最佳结果，您应始终在源和副本上使用相同的服务器 SQL 模式。

**性能考虑。** 分区操作对性能的一些影响如下：

+   **文件系统操作。** 分区和重新分区操作（例如`ALTER TABLE` 使用 `PARTITION BY ...`, `REORGANIZE PARTITION`, 或 `REMOVE PARTITIONING`）的实现取决于文件系统操作。这意味着这些操作的速度受到诸如文件系统类型和特性、磁盘速度、交换空间、操作系统的文件处理效率以及与文件处理相关的 MySQL 服务器选项和变量的影响。特别是，您应确保 `large_files_support` 已启用，并且 `open_files_limit` 已正确设置。涉及 `InnoDB` 表的分区和重新分区操作可能通过启用 `innodb_file_per_table` 变得更加高效。

    另请参阅 最大分区数。

+   **表锁。** 通常，对表执行分区操作的进程会对表进行写锁定。对这些表的读取相对不受影响；挂起的 `INSERT` 和 `UPDATE` 操作将在分区操作完成后立即执行。有关 `InnoDB` 特定的例外情况，请参阅 分区操作。

+   **索引；分区修剪。** 与非分区表一样，正确使用索引可以显着加快对分区表的查询速度。此外，设计分区表和对这些表的查询以利用分区修剪可以显著提高性能。有关更多信息，请参阅第 26.4 节，“分区修剪”。

    支持对分区表进行索引条件下推。请参阅第 10.2.1.6 节，“索引条件下推优化”。

+   **使用 LOAD DATA 的性能。** 在 MySQL 8.0 中，`LOAD DATA`使用缓冲区来提高性能。您应该知道，为了实现这一点，每个分区的缓冲区使用 130 KB 内存。

**最大分区数。** 对于不使用`NDB`存储引擎的给定表，最大可能的分区数为 8192。此数字包括子分区。

对于使用`NDB`存储引擎的表的最大可能的用户定义分区数根据使用的 NDB Cluster 软件版本、数据节点数量和其他因素确定。有关更多信息，请参阅 NDB 和用户定义分区。

如果在创建具有大量分区的表（但少于最大值）时遇到错误消息，例如从存储引擎获取错误...：打开文件时资源不足，您可以通过增加`open_files_limit`系统变量的值来解决此问题。但是，这取决于操作系统，并且在所有平台上可能不可行或不可取；有关更多信息，请参阅第 B.3.2.16 节，“文件未找到和类似错误”。在某些情况下，由于其他原因，使用大量（数百个）分区也可能不可取，因此使用更多分区并不会自动导致更好的结果。

另请参阅文件系统操作。

**不支持对分区 InnoDB 表使用外键。** 使用`InnoDB`存储引擎的分区表不支持外键。更具体地说，这意味着以下两个语句是正确的：

1.  不得包含外键引用的`InnoDB`表的定义使用用户定义的分区；包含外键引用的`InnoDB`表的定义不得分区。

1.  任何`InnoDB`表定义都不能包含对用户分区表的外键引用；任何具有用户定义分区的`InnoDB`表都不能包含被外键引用的列。

刚刚列出的限制范围包括所有使用`InnoDB`存储引擎的表。不允许创建`CREATE TABLE`和`ALTER TABLE`语句导致违反这些限制的表。

**ALTER TABLE ... ORDER BY。** 对分区表运行的`ALTER TABLE ... ORDER BY *`column`*`语句仅导致每个分区内的行排序。

**ADD COLUMN ... ALGORITHM=INSTANT。** 一旦在分区表上执行`ALTER TABLE ... ADD COLUMN ... ALGORITHM=INSTANT`，就不再可能与该表交换分区。

**通过修改主键对 REPLACE 语句的影响。** 在某些情况下可能是可取的（参见第 26.6.1 节，“分区键、主键和唯一键”），修改表的主键。请注意，如果您的应用程序使用`REPLACE`语句并且您这样做，这些语句的结果可能会发生 drastical 改变。有关更多信息和示例，请参见第 15.2.12 节，“REPLACE 语句”。

**全文索引。** 分区表不支持`FULLTEXT`索引或搜索。

**空间列。** 具有空间数据类型（如`POINT`或`GEOMETRY`）的列不能在分区表中使用。

**临时表。** 临时表不能被分区。

**日志表。** 无法对日志表进行分区；对这样的表运行`ALTER TABLE ... PARTITION BY ...`语句会失败并显示错误。

**分区键的数据类型。** 分区键必须是整数列或解析为整数的表达式。不能使用包含`ENUM`列的表达式。列或表达式的值也可以是`NULL`；请参见第 26.2.7 节，“MySQL 分区如何处理 NULL”。

这个限制有两个例外：

1.  当通过[`LINEAR`] `KEY`进行分区时，可以使用除`TEXT`或`BLOB`之外的任何有效的 MySQL 数据类型作为分区键，因为内部键哈希函数会从这些类型中生成正确的数据类型。例如，以下两个`CREATE TABLE`语句是有效的：

    ```sql
    CREATE TABLE tkc (c1 CHAR)
    PARTITION BY KEY(c1)
    PARTITIONS 4;

    CREATE TABLE tke
        ( c1 ENUM('red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet') )
    PARTITION BY LINEAR KEY(c1)
    PARTITIONS 6;
    ```

1.  当使用`RANGE COLUMNS`或`LIST COLUMNS`进行分区时，可以使用字符串、`DATE`和`DATETIME`列。例如，以下每个`CREATE TABLE`语句都是有效的：

    ```sql
    CREATE TABLE rc (c1 INT, c2 DATE)
    PARTITION BY RANGE COLUMNS(c2) (
        PARTITION p0 VALUES LESS THAN('1990-01-01'),
        PARTITION p1 VALUES LESS THAN('1995-01-01'),
        PARTITION p2 VALUES LESS THAN('2000-01-01'),
        PARTITION p3 VALUES LESS THAN('2005-01-01'),
        PARTITION p4 VALUES LESS THAN(MAXVALUE)
    );

    CREATE TABLE lc (c1 INT, c2 CHAR(1))
    PARTITION BY LIST COLUMNS(c2) (
        PARTITION p0 VALUES IN('a', 'd', 'g', 'j', 'm', 'p', 's', 'v', 'y'),
        PARTITION p1 VALUES IN('b', 'e', 'h', 'k', 'n', 'q', 't', 'w', 'z'),
        PARTITION p2 VALUES IN('c', 'f', 'i', 'l', 'o', 'r', 'u', 'x', NULL)
    );
    ```

前述两个例外情况均不适用于`BLOB`或`TEXT`列类型。

**子查询。** 分区键不能是子查询，即使该子查询解析为整数值或`NULL`。

**不支持列索引前缀用于键分区。** 在创建按键分区的表时，分区键中使用列前缀的任何列都不会用于表的分区函数。考虑以下`CREATE TABLE`语句，其中有三个`VARCHAR`列，主键使用了所有三列，并为其中两列指定了前缀：

```sql
CREATE TABLE t1 (
    a VARCHAR(10000),
    b VARCHAR(25),
    c VARCHAR(10),
    PRIMARY KEY (a(10), b, c(2))
) PARTITION BY KEY() PARTITIONS 2;
```

此语句被接受，但实际创建的表实际上是如同您发出了以下语句一样创建的，仅使用了不包含前缀的主键列（列`b`）作为分区键：

```sql
CREATE TABLE t1 (
    a VARCHAR(10000),
    b VARCHAR(25),
    c VARCHAR(10),
    PRIMARY KEY (a(10), b, c(2))
) PARTITION BY KEY(b) PARTITIONS 2;
```

在 MySQL 8.0.21 之前，没有发出警告或提供任何其他指示表明发生了这种情况，除非所有指定为分区键的列都使用了前缀，此时语句会失败，但会显示一个误导性的错误消息，如下所示：

```sql
mysql> CREATE TABLE t2 (
 ->     a VARCHAR(10000),
 ->     b VARCHAR(25),
 ->     c VARCHAR(10),
 ->     PRIMARY KEY (a(10), b(5), c(2))
 -> ) PARTITION BY KEY() PARTITIONS 2;
ERROR 1503 (HY000): A PRIMARY KEY must include all columns in the
table's partitioning function
```

当执行`ALTER TABLE`或升级此类表时也会发生这种情况。

从 MySQL 8.0.21 开始，不再支持这种宽松行为（并且可能在将来的 MySQL 版本中删除）。从 MySQL 8.0.21 开始，使用一个或多个具有前缀的列作为分区键会对每个这样的列产生警告，如下所示：

```sql
mysql> CREATE TABLE t1 (
 ->     a VARCHAR(10000),
 ->     b VARCHAR(25),
 ->     c VARCHAR(10),
 ->     PRIMARY KEY (a(10), b, c(2))
 -> ) PARTITION BY KEY() PARTITIONS 2;
Query OK, 0 rows affected, 2 warnings (1.25 sec)

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Warning
   Code: 1681
Message: Column 'test.t1.a' having prefix key part 'a(10)' is ignored by the
partitioning function. Use of prefixed columns in the PARTITION BY KEY() clause
is deprecated and will be removed in a future release.
*************************** 2\. row ***************************
  Level: Warning
   Code: 1681
Message: Column 'test.t1.c' having prefix key part 'c(2)' is ignored by the
partitioning function. Use of prefixed columns in the PARTITION BY KEY() clause
is deprecated and will be removed in a future release. 2 rows in set (0.00 sec)
```

这包括分区函数中使用的列被隐式定义为通过使用空的`PARTITION BY KEY()`子句来定义表的主键中的列的情况。

在 MySQL 8.0.21 及更高版本中，如果分区键指定的所有列都使用了前缀，则使用的`CREATE TABLE`语句将因错误消息而失败，该消息正确标识了问题：

```sql
mysql> CREATE TABLE t1 (
 ->     a VARCHAR(10000),
 ->     b VARCHAR(25),
 ->     c VARCHAR(10),
 ->     PRIMARY KEY (a(10), b(5), c(2))
 -> ) PARTITION BY KEY() PARTITIONS 2;
ERROR 1503 (HY000): A PRIMARY KEY must include all columns in the table's
partitioning function (prefixed columns are not considered).
```

有关按键分区表的一般信息，请参见第 26.2.5 节，“键分区”。

**子分区存在问题。** 子分区必须使用`HASH`或`KEY`分区。只有`RANGE`和`LIST`分区可以进行子分区；`HASH`和`KEY`分区不能进行子分区。

`SUBPARTITION BY KEY`要求明确指定子分区列或列，不同于`PARTITION BY KEY`的情况，后者可以省略（在这种情况下，默认使用表的主键列）。考虑通过此语句创建的表：

```sql
CREATE TABLE ts (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(30)
);
```

您可以通过类似于以下语句创建一个具有相同列、按`KEY`分区的表：

```sql
CREATE TABLE ts (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(30)
)
PARTITION BY KEY()
PARTITIONS 4;
```

前面的语句被视为如下所示编写，其中表的主键列用作分区列：

```sql
CREATE TABLE ts (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(30)
)
PARTITION BY KEY(id)
PARTITIONS 4;
```

然而，尝试使用默认列作为子分区列创建子分区表的以下语句将失败，必须为语句指定列才能成功，如下所示：

```sql
mysql> CREATE TABLE ts (
 ->     id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
 ->     name VARCHAR(30)
 -> )
 -> PARTITION BY RANGE(id)
 -> SUBPARTITION BY KEY()
 -> SUBPARTITIONS 4
 -> (
 ->     PARTITION p0 VALUES LESS THAN (100),
 ->     PARTITION p1 VALUES LESS THAN (MAXVALUE)
 -> );
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that
corresponds to your MySQL server version for the right syntax to use near ') 
mysql> CREATE TABLE ts (
 ->     id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
 ->     name VARCHAR(30)
 -> )
 -> PARTITION BY RANGE(id)
 -> SUBPARTITION BY KEY(id)
 -> SUBPARTITIONS 4
 -> (
 ->     PARTITION p0 VALUES LESS THAN (100),
 ->     PARTITION p1 VALUES LESS THAN (MAXVALUE)
 -> );
Query OK, 0 rows affected (0.07 sec)
```

这是一个已知问题（参见 Bug #51470）。

**数据目录和索引目录选项。** 忽略表级`DATA DIRECTORY`和`INDEX DIRECTORY`选项（参见 Bug #32091）。您可以为`InnoDB`表的各个分区或子分区使用这些选项。截至 MySQL 8.0.21，`DATA DIRECTORY`子句中指定的目录必须为`InnoDB`所知。有关更多信息，请参见使用 DATA DIRECTORY 子句。

**修复和重建分区表。** `CHECK TABLE`、`OPTIMIZE TABLE`、`ANALYZE TABLE`和`REPAIR TABLE`语句支持分区表。

此外，您可以使用`ALTER TABLE ... REBUILD PARTITION`来重建分区表的一个或多个分区；`ALTER TABLE ... REORGANIZE PARTITION`也会导致分区重建。有关这两个语句的更多信息，请参见第 15.1.9 节，“ALTER TABLE Statement”。

`ANALYZE`、`CHECK`、`OPTIMIZE`、`REPAIR`和`TRUNCATE`操作支持子分区。请参见第 15.1.9.1 节，“ALTER TABLE Partition Operations”。

**用于分区和子分区的文件名分隔符。** 表分区和子分区文件名包括生成的分隔符，如`#P#`和`#SP#`。此类分隔符的大小写可能有所不同，不应依赖于此。
