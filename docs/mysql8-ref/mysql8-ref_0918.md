> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-table-ndb-comment-options.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table-ndb-comment-options.html)

#### 15.1.20.12 设置 NDB 注释选项

+   NDB_COLUMN 选项

+   NDB_TABLE 选项

可以在 `NDB` 表的表注释或列注释中设置许多特定于 NDB Cluster 的选项。 通过使用 `NDB_TABLE` 在表注释中嵌入用于控制从任何副本读取和分区平衡的表级选项。

`NDB_COLUMN` 可以用于列注释，将 `NDB` 用于存储 blob 值的 blob 部分表列的大小设置为最大值。 这适用于 `BLOB`、`MEDIUMBLOB`、`LONGBLOB`、`TEXT`、`MEDIUMTEXT`、`LONGTEXT` 和 `JSON` 列。 从 NDB 8.0.30 开始，列注释还可以用于控制 blob 列的内联大小。 `NDB_COLUMN` 注释不支持 `TINYBLOB` 或 `TINYTEXT` 列，因为这些列具有固定大小的内联部分（仅）和无法存储在其他位置的单独部分。

`NDB_TABLE` 可以用于表注释，用于设置与分区平衡和表是否完全复制等相关的选项。

本节的其余部分描述了这些选项及其用法。

##### NDB_COLUMN 选项

在 NDB Cluster 中，`CREATE TABLE` 或 `ALTER TABLE` 语句中的列注释也可以用于指定 `NDB_COLUMN` 选项。从版本 8.0.30 开始，`NDB` 支持两个列注释选项 `BLOB_INLINE_SIZE` 和 `MAX_BLOB_PART_SIZE`。 （在 NDB 8.0.30 之前，仅支持 `MAX_BLOB_PART_SIZE`。）此选项的语法如下所示：

```sql
COMMENT 'NDB_COLUMN=*speclist*'

*speclist* := *spec*[,*spec*]

spec := 
    BLOB_INLINE_SIZE=*value*
  | MAX_BLOB_PART_SIZE[={0|1}]
```

`BLOB_INLINE_SIZE` 指定要由该列内联存储的字节数；其预期值为 1 - 29980 范围内的整数。 设置大于 29980 的值会引发错误； 允许设置小于 1 的值，但会导致使用列类型的默认内联大小。

您应该知道，此选项的最大值实际上是可以存储在一个 `NDB` 表的一行中的最大字节数； 行中的每列都会对此总数做出贡献。

您还应该记住，特别是在处理 `TEXT` 列时，由 `MAX_BLOB_PART_SIZE` 或 `BLOB_INLINE_SIZE` 设置的值表示列大小（以字节为单位）。 它不表示字符数，字符数根据列使用的字符集和排序规则而变化。

要查看此选项的效果，请首先创建一个具有两个`BLOB`列的表，一个（`b1`）没有额外选项，另一个（`b2`）设置了`BLOB_INLINE_SIZE`，如下所示：

```sql
mysql> CREATE TABLE t1 (
 ->    a INT NOT NULL PRIMARY KEY,
 ->    b1 BLOB,
 ->    b2 BLOB COMMENT 'NDB_COLUMN=BLOB_INLINE_SIZE=8000'
 ->  ) ENGINE NDB;
Query OK, 0 rows affected (0.32 sec)
```

您可以通过查询`ndbinfo.blobs`表来查看`BLOB`列的`BLOB_INLINE_SIZE`设置，如下所示：

```sql
mysql> SELECT 
 ->   column_name AS 'Column Name', 
 ->   inline_size AS 'Inline Size', 
 ->   part_size AS 'Blob Part Size' 
 -> FROM ndbinfo.blobs 
 -> WHERE table_name = 't1';
+-------------+-------------+----------------+
| Column Name | Inline Size | Blob Part Size |
+-------------+-------------+----------------+
| b1          |         256 |           2000 |
| b2          |        8000 |           2000 |
+-------------+-------------+----------------+
2 rows in set (0.01 sec)
```

您还可以检查**ndb_desc**实用程序的输出，如下所示，相关行以强调文本显示：

```sql
$> ndb_desc -d test t1
-- t --
Version: 1
Fragment type: HashMapPartition
K Value: 6
Min load factor: 78
Max load factor: 80
Temporary table: no
Number of attributes: 3
Number of primary keys: 1
Length of frm data: 945
Max Rows: 0
Row Checksum: 1
Row GCI: 1
SingleUserMode: 0
ForceVarPart: 1
PartitionCount: 2
FragmentCount: 2
PartitionBalance: FOR_RP_BY_LDM
ExtraRowGciBits: 0
ExtraRowAuthorBits: 0
TableStatus: Retrieved
Table options: readbackup
HashMap: DEFAULT-HASHMAP-3840-2
-- Attributes --
a Int PRIMARY KEY DISTRIBUTION KEY AT=FIXED ST=MEMORY
*b1 Blob(256,2000,0) NULL AT=MEDIUM_VAR ST=MEMORY BV=2 BT=NDB$BLOB_64_1
b2 Blob(8000,2000,0) NULL AT=MEDIUM_VAR ST=MEMORY BV=2 BT=NDB$BLOB_64_2* -- Indexes -- 
PRIMARY KEY(a) - UniqueHashIndex
PRIMARY(a) - OrderedIndex
```

对于`MAX_BLOB_PART_SIZE`，`=`号和其后的值是可选的。使用除 0 或 1 之外的任何值会导致语法错误。

在列注释中使用`MAX_BLOB_PART_SIZE`的效果是将`TEXT`或`BLOB`列的 blob 部分大小设置为`NDB`支持的最大字节数（13948）。此选项可应用于 MySQL 支持的除`TINYBLOB`或`TINYTEXT`之外的任何 blob 列类型（`BLOB`、`MEDIUMBLOB`、`LONGBLOB`、`TEXT`、`MEDIUMTEXT`、`LONGTEXT`）。与`BLOB_INLINE_SIZE`不同，`MAX_BLOB_PART_SIZE`对`JSON`列没有影响。

要查看此选项的效果，我们首先在**mysql**客户端中运行以下 SQL 语句，创建一个具有两个`BLOB`列的表，一个（`c1`）没有额外选项，另一个（`c2`）具有`MAX_BLOB_PART_SIZE`：

```sql
mysql> CREATE TABLE test.t2 (
 ->   p INT PRIMARY KEY, 
 ->   c1 BLOB, 
 ->   c2 BLOB COMMENT 'NDB_COLUMN=MAX_BLOB_PART_SIZE'
 -> ) ENGINE NDB;
Query OK, 0 rows affected (0.32 sec)
```

从系统 shell 中运行**ndb_desc**实用程序，以获取有关刚刚创建的表的信息，如此示例所示：

```sql
$> ndb_desc -d test t2
-- t --
Version: 1
Fragment type: HashMapPartition
K Value: 6
Min load factor: 78
Max load factor: 80
Temporary table: no
Number of attributes: 3
Number of primary keys: 1
Length of frm data: 324
Row Checksum: 1
Row GCI: 1
SingleUserMode: 0
ForceVarPart: 1
FragmentCount: 2
ExtraRowGciBits: 0
ExtraRowAuthorBits: 0
TableStatus: Retrieved
HashMap: DEFAULT-HASHMAP-3840-2
-- Attributes --
p Int PRIMARY KEY DISTRIBUTION KEY AT=FIXED ST=MEMORY
*c1 Blob(256,2000,0) NULL AT=MEDIUM_VAR ST=MEMORY BV=2 BT=NDB$BLOB_22_1
c2 Blob(256,13948,0) NULL AT=MEDIUM_VAR ST=MEMORY BV=2 BT=NDB$BLOB_22_2* -- Indexes -- 
PRIMARY KEY(p) - UniqueHashIndex
PRIMARY(p) - OrderedIndex
```

输出中的列信息在`Attributes`下列出；对于`c1`和`c2`列，它在这里以强调文本显示。对于`c1`，blob 部分大小为 2000，即默认值；对于`c2`，它为 13948，由`MAX_BLOB_PART_SIZE`设置。

您还可以查询`ndbinfo.blobs`表来查看此内容，如下所示：

```sql
mysql> SELECT 
 ->   column_name AS 'Column Name', 
 ->   inline_size AS 'Inline Size', 
 ->   part_size AS 'Blob Part Size' 
 -> FROM ndbinfo.blobs 
 -> WHERE table_name = 't2';
+-------------+-------------+----------------+
| Column Name | Inline Size | Blob Part Size |
+-------------+-------------+----------------+
| c1          |         256 |           2000 |
| c2          |         256 |          13948 |
+-------------+-------------+----------------+
2 rows in set (0.00 sec)
```

您可以使用类似于这样的`ALTER TABLE`语句更改`NDB`表的给定 blob 列的 blob 部分大小，并使用`SHOW CREATE TABLE`在之后验证更改：

```sql
mysql> ALTER TABLE test.t2 
 ->    DROP COLUMN c1, 
 ->     ADD COLUMN c1 BLOB COMMENT 'NDB_COLUMN=MAX_BLOB_PART_SIZE',
 ->     CHANGE COLUMN c2 c2 BLOB AFTER c1;
Query OK, 0 rows affected (0.47 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE test.t2\G
*************************** 1\. row ***************************
       Table: t
Create Table: CREATE TABLE `t2` (
  `p` int(11) NOT NULL,
  `c1` blob COMMENT 'NDB_COLUMN=MAX_BLOB_PART_SIZE',
  `c2` blob,
  PRIMARY KEY (`p`)
) ENGINE=ndbcluster DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.00 sec)

mysql> EXIT
Bye
```

**ndb_desc**的输出显示列的 blob 部分大小已按预期更改：

```sql
$> ndb_desc -d test t2
-- t --
Version: 16777220
Fragment type: HashMapPartition
K Value: 6
Min load factor: 78
Max load factor: 80
Temporary table: no
Number of attributes: 3
Number of primary keys: 1
Length of frm data: 324
Row Checksum: 1
Row GCI: 1
SingleUserMode: 0
ForceVarPart: 1
FragmentCount: 2
ExtraRowGciBits: 0
ExtraRowAuthorBits: 0
TableStatus: Retrieved
HashMap: DEFAULT-HASHMAP-3840-2
-- Attributes --
p Int PRIMARY KEY DISTRIBUTION KEY AT=FIXED ST=MEMORY
*c1 Blob(256,13948,0) NULL AT=MEDIUM_VAR ST=MEMORY BV=2 BT=NDB$BLOB_26_1
c2 Blob(256,2000,0) NULL AT=MEDIUM_VAR ST=MEMORY BV=2 BT=NDB$BLOB_26_2* -- Indexes -- 
PRIMARY KEY(p) - UniqueHashIndex
PRIMARY(p) - OrderedIndex
```

您还可以通过再次运行针对`ndbinfo.blobs`的查询来查看更改：

```sql
mysql> SELECT 
 ->   column_name AS 'Column Name', 
 ->   inline_size AS 'Inline Size', 
 ->   part_size AS 'Blob Part Size' 
 -> FROM ndbinfo.blobs 
 -> WHERE table_name = 't2';
+-------------+-------------+----------------+
| Column Name | Inline Size | Blob Part Size |
+-------------+-------------+----------------+
| c1          |         256 |          13948 |
| c2          |         256 |           2000 |
+-------------+-------------+----------------+
2 rows in set (0.00 sec)
```

可以为 blob 列同时设置`BLOB_INLINE_SIZE`和`MAX_BLOB_PART_SIZE`，如此`CREATE TABLE`语句所示：

```sql
mysql> CREATE TABLE test.t3 (
 ->   p INT NOT NULL PRIMARY KEY,
 ->   c1 JSON,
 ->   c2 JSON COMMENT 'NDB_COLUMN=BLOB_INLINE_SIZE=5000,MAX_BLOB_PART_SIZE'
 -> ) ENGINE NDB;
Query OK, 0 rows affected (0.28 sec)
```

查询`blobs`表显示该语句按预期工作：

```sql
mysql> SELECT 
 ->   column_name AS 'Column Name', 
 ->   inline_size AS 'Inline Size', 
 ->   part_size AS 'Blob Part Size' 
 -> FROM ndbinfo.blobs 
 -> WHERE table_name = 't3';
+-------------+-------------+----------------+
| Column Name | Inline Size | Blob Part Size |
+-------------+-------------+----------------+
| c1          |        4000 |           8100 |
| c2          |        5000 |           8100 |
+-------------+-------------+----------------+
2 rows in set (0.00 sec)
```

您还可以通过检查 **ndb_desc** 的输出来验证该语句是否有效。

更改列的 blob 部分大小必须使用复制的 `ALTER TABLE` 来完成；此操作无法在线执行（参见 第 25.6.12 节，“NDB 集群中的 ALTER TABLE 在线操作”）。

有关 `NDB` 如何存储 blob 类型列的更多信息，请参阅 字符串类型存储要求。

##### NDB_TABLE 选项

对于 NDB 集群表，在 `CREATE TABLE` 或 `ALTER TABLE` 语句中的表注释也可以用于指定一个 `NDB_TABLE` 选项，该选项由一个或多个名称-值对组成，如果需要，用逗号分隔，跟在字符串 `NDB_TABLE=` 后面。名称和值的完整语法如下所示：

```sql
COMMENT="NDB_TABLE=*ndb_table_option*[,*ndb_table_option*[,...]]"

*ndb_table_option*: {
    NOLOGGING={1 | 0}
  | READ_BACKUP={1 | 0}
  | PARTITION_BALANCE={FOR_RP_BY_NODE | FOR_RA_BY_NODE | FOR_RP_BY_LDM
                      | FOR_RA_BY_LDM | FOR_RA_BY_LDM_X_2
                      | FOR_RA_BY_LDM_X_3 | FOR_RA_BY_LDM_X_4}
  | FULLY_REPLICATED={1 | 0}
}
```

引号字符串内不允许有空格。字符串不区分大小写。

可以通过表注释中的四个 `NDB` 表选项来设置这种方式。这几个选项将在接下来的几段中详细描述。

`NOLOGGING`：默认情况下，`NDB` 表是记录并进行检查点的。这使得它们在整个集群失败时是持久的。在创建或更改表时使用 `NOLOGGING` 意味着该表不会被重做日志记录或包含在本地检查点中。在这种情况下，该表仍然在数据节点之间复制以实现高可用性，并使用事务进行更新，但对其所做的更改不会记录在数据节点的重做日志中，并且其内容不会被检查点到磁盘上；在从集群故障中恢复时，集群会保留表定义，但不保留任何行，也就是说，该表是空的。

使用这种非记录表减少了数据节点对磁盘 I/O 和存储的需求，以及用于检查点的 CPU。这可能适用于频繁更新的短期数据，并且在极少数情况下发生总集群故障时可以接受所有数据的丢失。

还可以使用 `ndb_table_no_logging` 系统变量，使得在此变量生效时创建或更改的任何 NDB 表都表现得好像已经使用 `NOLOGGING` 注释创建。与直接使用注释时不同，在 `SHOW CREATE TABLE` 的输出中没有任何内容表明它是一个非记录表。推荐使用表注释方法，因为它提供了对该功能的每个表的控制，并且表模式的这一方面嵌入在表创建语句中，可以很容易地被 SQL 工具找到。

`READ_BACKUP`: 将此选项设置为 1 的效果与启用`ndb_read_backup`相同；允许从任何副本读取。这样做极大地提高了从表中读取的性能，对写入性能的影响相对较小。从 NDB 8.0.19 开始，`READ_BACKUP`的默认值为 1，`ndb_read_backup`的默认值为`ON`（以前，默认情况下，从任何副本读取是禁用的）。

您可以在线为现有表设置`READ_BACKUP`，使用类似于以下示例的`ALTER TABLE`语句：

```sql
ALTER TABLE ... ALGORITHM=INPLACE, COMMENT="NDB_TABLE=READ_BACKUP=1";

ALTER TABLE ... ALGORITHM=INPLACE, COMMENT="NDB_TABLE=READ_BACKUP=0";
```

有关`ALTER TABLE`的`ALGORITHM`选项的更多信息，请参阅第 25.6.12 节，“NDB Cluster 中的 ALTER TABLE 在线操作”。

`PARTITION_BALANCE`: 提供对分区分配和放置的额外控制。支持以下四种方案：

1.  `FOR_RP_BY_NODE`: 每个节点一个分区。

    每个节点上只有一个 LDM 存储主分区。每个分区在所有节点上的相同 LDM（相同 ID）中存储。

1.  `FOR_RA_BY_NODE`: 每个节点组一个分区。

    每个节点存储一个分区，可以是主复制品或备份复制品。每个分区在所有节点上都存储在相同的 LDM 中。

1.  `FOR_RP_BY_LDM`: 每个节点上每个 LDM 一个分区；默认设置。

    如果将`READ_BACKUP`设置为 1，则使用此设置。

1.  `FOR_RA_BY_LDM`: 每个节点组中每个 LDM 一个分区。

    这些分区可以是主分区或备份分区。

1.  `FOR_RA_BY_LDM_X_2`: 每个节点组中每个 LDM 有两个分区。

    这些分区可以是主分区或备份分区。

1.  `FOR_RA_BY_LDM_X_3`: 每个节点组中每个 LDM 有三个分区。

    这些分区可以是主分区或备份分区。

1.  `FOR_RA_BY_LDM_X_4`: 每个节点组中每个 LDM 有四个分区。

    这些分区可以是主分区或备份分区。

`PARTITION_BALANCE`是设置每个表分区数的首选接口。使用`MAX_ROWS`强制分区数已被弃用，但仍继续支持以确保向后兼容性；它可能在将来的 MySQL NDB Cluster 版本中被移除（Bug #81759，Bug #23544301）。

`FULLY_REPLICATED`控制表是否完全复制，即每个数据节点是否有表的完整副本。要启用表的完全复制，使用`FULLY_REPLICATED=1`。

这个设置也可以使用`ndb_fully_replicated`系统变量来控制。将其设置为`ON`会默认为所有新的`NDB`表启用该选项；默认值为`OFF`。`ndb_data_node_neighbour`系统变量也用于完全复制的表，以确保访问完全复制的表时，我们访问与此 MySQL 服务器本地的数据节点。

下面显示了创建`NDB`表时使用此类注释的`CREATE TABLE`语句示例：

```sql
mysql> CREATE TABLE t1 (
     >     c1 INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
     >     c2 VARCHAR(100),
     >     c3 VARCHAR(100) )
     > ENGINE=NDB
     >
COMMENT="NDB_TABLE=READ_BACKUP=0,PARTITION_BALANCE=FOR_RP_BY_NODE";
```

该注释显示为`SHOW CREATE TABLE`的输出的一部分。注释的文本也可以通过查询 MySQL 信息模式`TABLES`表来获取，例如：

```sql
mysql> SELECT TABLE_NAME, TABLE_SCHEMA, TABLE_COMMENT
     > FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME="t1"\G
*************************** 1\. row ***************************
   TABLE_NAME: t1
 TABLE_SCHEMA: test
TABLE_COMMENT: NDB_TABLE=READ_BACKUP=0,PARTITION_BALANCE=FOR_RP_BY_NODE 1 row in set (0.01 sec)
```

此注释语法也适用于`NDB`表的`ALTER TABLE`语句，如下所示：

```sql
mysql> ALTER TABLE t1 COMMENT="NDB_TABLE=PARTITION_BALANCE=FOR_RA_BY_NODE";
Query OK, 0 rows affected (0.40 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

从 NDB 8.0.21 开始，`TABLE_COMMENT`列显示了在`ALTER TABLE`语句后重新创建表所需的注释，如下所示：

```sql
mysql> SELECT TABLE_NAME, TABLE_SCHEMA, TABLE_COMMENT
 ->     FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME="t1"\G
*************************** 1\. row ***************************
   TABLE_NAME: t1
 TABLE_SCHEMA: test
TABLE_COMMENT: NDB_TABLE=READ_BACKUP=0,PARTITION_BALANCE=FOR_RP_BY_NODE 1 row in set (0.01 sec)
```

```sql
mysql> SELECT TABLE_NAME, TABLE_SCHEMA, TABLE_COMMENT
     > FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME="t1";
+------------+--------------+--------------------------------------------------+
| TABLE_NAME | TABLE_SCHEMA | TABLE_COMMENT                                    |
+------------+--------------+--------------------------------------------------+
| t1         | c            | NDB_TABLE=PARTITION_BALANCE=FOR_RA_BY_NODE       |
| t1         | d            |                                                  |
+------------+--------------+--------------------------------------------------+
2 rows in set (0.01 sec)
```

请记住，与`ALTER TABLE`一起使用的表注释会替换表可能已有的任何现有注释。

```sql
mysql> ALTER TABLE t1 COMMENT="NDB_TABLE=PARTITION_BALANCE=FOR_RA_BY_NODE";
Query OK, 0 rows affected (0.40 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SELECT TABLE_NAME, TABLE_SCHEMA, TABLE_COMMENT
     > FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME="t1";
+------------+--------------+--------------------------------------------------+
| TABLE_NAME | TABLE_SCHEMA | TABLE_COMMENT                                    |
+------------+--------------+--------------------------------------------------+
| t1         | c            | NDB_TABLE=PARTITION_BALANCE=FOR_RA_BY_NODE       |
| t1         | d            |                                                  |
+------------+--------------+--------------------------------------------------+
2 rows in set (0.01 sec)
```

在 NDB 8.0.21 之前，与`ALTER TABLE`一起使用的表注释会替换表可能已有的任何现有注释。这意味着（例如）`READ_BACKUP`值不会传递到由`ALTER TABLE`语句设置的新注释中，并且任何未指定的值都会恢复为默认值。（BUG#30428829）因此，使用 SQL 不再有任何方法来检索先前为注释设置的值。为了防止注释值恢复为默认值，需要保留现有注释字符串中的任何此类值，并将它们包含在传递给`ALTER TABLE`的注释中。

在**ndb_desc**的输出中，您还可以看到`PARTITION_BALANCE`选项的价值。**ndb_desc**还显示了表中是否设置了`READ_BACKUP`和`FULLY_REPLICATED`选项。有关此程序的更多信息，请参阅其描述。
