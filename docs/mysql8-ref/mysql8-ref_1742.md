> 译文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-disk-data-objects.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-disk-data-objects.html)

#### 25.6.11.1 NDB Cluster 磁盘数据对象

NDB Cluster 磁盘数据存储是使用以下对象实现的：

+   表空间：作为其他磁盘数据对象的容器。一个表空间包含一个或多个数据文件和一个或多个撤销日志文件组。

+   数据文件：存储列数据。数据文件直接分配给表空间。

+   撤销日志文件：包含用于回滚事务所需的撤销信息。分配给一个撤销日志文件组。

+   日志文件组：包含一个或多个撤销日志文件。分配给一个表空间。

撤销日志文件和数据文件是每个数据节点文件系统中的实际文件；默认情况下，它们放置在`ndb_*`node_id`*_fs`中，在 NDB Cluster `config.ini`文件中指定的*`DataDir`*中，其中*`node_id`*是数据节点的节点 ID。可以通过在创建撤销日志或数据文件时指定绝对或相对路径的方式将其放置在其他位置。创建这些文件的语句稍后在本节中显示。

撤销日志文件仅用于磁盘数据表，并且不需要或不用于仅存储在内存中的`NDB`表。

NDB Cluster 表空间和日志文件组并非实现为文件。

尽管并非所有磁盘数据对象都实现为文件，但它们都共享相同的命名空间。这意味着*每个磁盘数据对象*必须具有唯一的名称（而不仅仅是给定类型的每个磁盘数据对象）。例如，您不能同时命名一个表空间和一个日志文件组为`dd1`。

假设您已经设置好了包括管理节点和 SQL 节点在内的 NDB Cluster，创建 NDB Cluster 磁盘上的表的基本步骤如下：

1.  创建一个日志文件组，并将一个或多个撤销日志文件分配给它（有时也称为撤销日志文件）。

1.  创建一个表空间；将日志文件组以及一个或多个数据文件分配给表空间。

1.  创建一个使用此表空间进行数据存储的磁盘数据表。

可以使用 SQL 语句在**mysql**客户端或其他 MySQL 客户端应用程序中完成这些任务，如下面的示例所示。

1.  我们使用`CREATE LOGFILE GROUP`创建一个名为`lg_1`的日志文件组。该日志文件组由两个撤销日志文件组成，我们将它们命名为`undo_1.log`和`undo_2.log`，它们的初始大小分别为 16 MB 和 12 MB。（撤销日志文件的默认初始大小为 128 MB。）可选地，您还可以为日志文件组的撤销缓冲区指定大小，或允许其采用默认值 8 MB。在本例中，我们将 UNDO 缓冲区的大小设置为 2 MB。必须使用撤销日志文件创建日志文件组；因此，在此`CREATE LOGFILE GROUP`语句中，我们将`undo_1.log`添加到`lg_1`中：

    ```sql
    CREATE LOGFILE GROUP lg_1
        ADD UNDOFILE 'undo_1.log'
        INITIAL_SIZE 16M
        UNDO_BUFFER_SIZE 2M
        ENGINE NDBCLUSTER;
    ```

    要将`undo_2.log`添加到日志文件组中，请使用以下`ALTER LOGFILE GROUP`语句：

    ```sql
    ALTER LOGFILE GROUP lg_1
        ADD UNDOFILE 'undo_2.log'
        INITIAL_SIZE 12M
        ENGINE NDBCLUSTER;
    ```

    一些需要注意的事项：

    +   此处使用的`.log`文件扩展名并非必需。我们仅使用它使日志文件易于识别。

    +   每个`CREATE LOGFILE GROUP`和`ALTER LOGFILE GROUP`语句必须包含一个`ENGINE`选项。此选项的唯一允许值为`NDBCLUSTER`和`NDB`。

        重要

        在同一个 NDB 集群中最多只能存在一个日志文件组。

    +   当您使用`ADD UNDOFILE '*`filename`*'`将撤销日志文件添加到日志文件组时，将在集群中每个数据节点的`ndb_*`node_id`*_fs`目录中创建一个名为*`filename`*的文件，其中*`node_id`*是数据节点的节点 ID。每个撤销日志文件的大小与 SQL 语句中指定的大小相同。例如，如果一个 NDB 集群有 4 个数据节点，则刚刚显示的`ALTER LOGFILE GROUP`语句将创建 4 个撤销日志文件，每个数据节点的数据目录中各有一个；每个文件的名称为`undo_2.log`，大小为 12 MB。

    +   `UNDO_BUFFER_SIZE`受系统可用内存量的限制。

    +   有关这些语句的更多信息，请参见 Section 15.1.16, “CREATE LOGFILE GROUP Statement”和 Section 15.1.6, “ALTER LOGFILE GROUP Statement”。

1.  现在我们可以创建一个表空间——用于存储数据的磁盘数据表使用的文件的抽象容器。表空间与特定的日志文件组相关联；在创建新表空间时，必须指定其用于撤销日志记录的日志文件组。还必须指定至少一个数据文件；在创建表空间后，可以向表空间添加更多数据文件。还可以从表空间中删除数据文件（请参见本节后面的示例）。

    假设我们希望创建一个名为`ts_1`的表空间，该表空间使用`lg_1`作为其日志文件组。我们希望表空间包含两个数据文件，分别命名为`data_1.dat`和`data_2.dat`，其初始大小分别为 32 MB 和 48 MB。（`INITIAL_SIZE`的默认值为 128 MB。）我们可以使用两个 SQL 语句来实现这一点，如下所示：

    ```sql
    CREATE TABLESPACE ts_1
        ADD DATAFILE 'data_1.dat'
        USE LOGFILE GROUP lg_1
        INITIAL_SIZE 32M
        ENGINE NDBCLUSTER;

    ALTER TABLESPACE ts_1
        ADD DATAFILE 'data_2.dat'
        INITIAL_SIZE 48M;
    ```

    `CREATE TABLESPACE`语句创建一个名为`ts_1`的表空间，其中包含数据文件`data_1.dat`，并将`ts_1`与日志文件组`lg_1`关联。`ALTER TABLESPACE`添加了第二个数据文件（`data_2.dat`）。

    一些需要注意的事项：

    +   就像在此示例中用于撤销日志文件的`.log`文件扩展名一样，在`.dat`文件扩展名中没有特殊意义；它仅用于便于识别。

    +   当您使用`ADD DATAFILE '*`filename`*'`向表空间添加数据文件时，将在集群中每个数据节点的`ndb_*`node_id`*_fs`目录中创建一个名为*`filename`*的文件，其中*`node_id`*是数据节点的节点 ID。每个数据文件的大小与 SQL 语句中指定的大小相同。例如，如果一个 NDB 集群有 4 个数据节点，则刚刚显示的`ALTER TABLESPACE`语句将创建 4 个数据文件，每个数据节点的数据目录中各有一个；每个文件的名称为`data_2.dat`，每个文件的大小为 48 MB。

    +   `NDB`为每个表空间保留 4%的空间，用于在数据节点重新启动期间使用。此空间不可用于存储数据。

    +   `CREATE TABLESPACE`语句必须包含一个`ENGINE`子句；只有使用与表空间相同存储引擎的表才能在表空间中创建。对于`ALTER TABLESPACE`，`ENGINE`子句被接受但已被弃用，并可能在将来的版本中被移除。对于`NDB`表空间，此选项的唯一允许值为`NDBCLUSTER`和`NDB`。

    +   在 NDB 8.0.20 及更高版本中，对于给定表空间使用的所有数据文件，范围的分配是以循环方式进行的。

    +   有关`CREATE TABLESPACE`和`ALTER TABLESPACE`语句的更多信息，请参见第 15.1.21 节，“CREATE TABLESPACE Statement”和第 15.1.10 节，“ALTER TABLESPACE Statement”。

1.  现在可以创建一个表，其未索引的列存储在使用表空间`ts_1`中的文件中：

    ```sql
    CREATE TABLE dt_1 (
        member_id INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
        last_name VARCHAR(50) NOT NULL,
        first_name VARCHAR(50) NOT NULL,
        dob DATE NOT NULL,
        joined DATE NOT NULL,
        INDEX(last_name, first_name)
        )
        TABLESPACE ts_1 STORAGE DISK
        ENGINE NDBCLUSTER;
    ```

    `TABLESPACE ts_1 STORAGE DISK`告诉`NDB`存储引擎在磁盘上使用表空间`ts_1`进行数据存储。

    一旦像所示创建了表`ts_1`，您可以执行`INSERT`、`SELECT`、`UPDATE`和`DELETE`语句，就像对任何其他 MySQL 表一样。

    通过在`CREATE TABLE`或`ALTER TABLE`语句中作为列定义的一部分使用`STORAGE`子句，还可以指定单个列是存储在磁盘上还是存储在内存中。`STORAGE DISK`导致列存储在磁盘上，而`STORAGE MEMORY`导致使用内存存储。有关更多信息，请参见第 15.1.20 节，“CREATE TABLE Statement”。

您可以通过查询`INFORMATION_SCHEMA`数据库中的`FILES`表来获取刚刚创建的`NDB`磁盘数据文件和撤销日志文件的信息，如下所示：

```sql
mysql> SELECT
              FILE_NAME AS File, FILE_TYPE AS Type,
              TABLESPACE_NAME AS Tablespace, TABLE_NAME AS Name,
              LOGFILE_GROUP_NAME AS 'File group',
              FREE_EXTENTS AS Free, TOTAL_EXTENTS AS Total
          FROM INFORMATION_SCHEMA.FILES
          WHERE ENGINE='ndbcluster';
+--------------+----------+------------+------+------------+------+---------+
| File         | Type     | Tablespace | Name | File group | Free | Total   |
+--------------+----------+------------+------+------------+------+---------+
| ./undo_1.log | UNDO LOG | lg_1       | NULL | lg_1       |    0 | 4194304 |
| ./undo_2.log | UNDO LOG | lg_1       | NULL | lg_1       |    0 | 3145728 |
| ./data_1.dat | DATAFILE | ts_1       | NULL | lg_1       |   32 |      32 |
| ./data_2.dat | DATAFILE | ts_1       | NULL | lg_1       |   48 |      48 |
+--------------+----------+------------+------+------------+------+---------+
4 rows in set (0.00 sec)
```

有关更多信息和示例，请参见第 28.3.15 节，“The INFORMATION_SCHEMA FILES Table”。

**隐式存储在磁盘上的列的索引。** 对于刚刚显示的示例中定义的表`dt_1`，只有`dob`和`joined`列存储在磁盘上。这是因为`id`、`last_name`和`first_name`列上有索引，因此属于这些列的数据存储在 RAM 中。只有非索引列可以存储在磁盘上；索引和索引列数据继续存储在内存中。在设计磁盘数据表时，您必须牢记索引使用和 RAM 保留之间的权衡。

不能向明确声明为`STORAGE DISK`的列添加索引，必须先将其存储类型更改为`MEMORY`；任何尝试这样做的操作都会失败并显示错误。可以对*隐式*使用磁盘存储的列进行索引；在这种情况下，列的存储类型会自动更改为`MEMORY`。所谓“隐式”，指的是存储类型未声明但是从父表继承的列。在以下 CREATE TABLE 语句中（使用先前定义的表空间`ts_1`），列`c2`和`c3`隐式使用磁盘存储：

```sql
mysql> CREATE TABLE ti (
 ->     c1 INT PRIMARY KEY,
 ->     c2 INT,
 ->     c3 INT,
 ->     c4 INT
 -> )
 ->     STORAGE DISK
 ->     TABLESPACE ts_1
 ->     ENGINE NDBCLUSTER;
Query OK, 0 rows affected (1.31 sec)
```

因为`c2`、`c3`和`c4`本身没有声明为`STORAGE DISK`，所以可以对它们进行索引。在这里，我们分别对`c2`和`c3`添加索引，使用`CREATE INDEX`和`ALTER TABLE`：

```sql
mysql> CREATE INDEX i1 ON ti(c2);
Query OK, 0 rows affected (2.72 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> ALTER TABLE ti ADD INDEX i2(c3);
Query OK, 0 rows affected (0.92 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

`SHOW CREATE TABLE`确认已添加索引。

```sql
mysql> SHOW CREATE TABLE ti\G
*************************** 1\. row ***************************
       Table: ti
Create Table: CREATE TABLE `ti` (
  `c1` int(11) NOT NULL,
  `c2` int(11) DEFAULT NULL,
  `c3` int(11) DEFAULT NULL,
  `c4` int(11) DEFAULT NULL,
  PRIMARY KEY (`c1`),
  KEY `i1` (`c2`),
  KEY `i2` (`c3`)
) /*!50100 TABLESPACE `ts_1` STORAGE DISK */ ENGINE=ndbcluster DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.00 sec)
```

可以使用**ndb_desc**查看，已强调的索引列现在使用内存而不是磁盘存储：

```sql
$> ./ndb_desc -d test t1
-- t1 --
Version: 33554433
Fragment type: HashMapPartition
K Value: 6
Min load factor: 78
Max load factor: 80
Temporary table: no
Number of attributes: 4
Number of primary keys: 1
Length of frm data: 317
Max Rows: 0
Row Checksum: 1
Row GCI: 1
SingleUserMode: 0
ForceVarPart: 1
PartitionCount: 4
FragmentCount: 4
PartitionBalance: FOR_RP_BY_LDM
ExtraRowGciBits: 0
ExtraRowAuthorBits: 0
TableStatus: Retrieved
Table options:
HashMap: DEFAULT-HASHMAP-3840-4
-- Attributes --
c1 Int PRIMARY KEY DISTRIBUTION KEY AT=FIXED ST=MEMORY
*c2 Int NULL AT=FIXED ST=MEMORY
c3 Int NULL AT=FIXED ST=MEMORY* c4 Int NULL AT=FIXED ST=DISK
-- Indexes --
PRIMARY KEY(c1) - UniqueHashIndex
i2(c3) - OrderedIndex
PRIMARY(c1) - OrderedIndex
i1(c2) - OrderedIndex
```

**性能注意。** 使用磁盘数据存储的集群的性能会大大提高，如果将磁盘数据文件与数据节点文件系统分开存放。必须为集群中的每个数据节点执行此操作才能获得任何明显的好处。

可以在`ADD UNDOFILE`和`ADD DATAFILE`中使用绝对和相对文件系统路径；相对路径是相对于数据节点的数据目录计算的。

日志文件组、表空间以及使用这些对象的任何磁盘数据表必须按特定顺序创建。对于删除这些对象也是如此，受以下约束条件限制：

+   只要任何表空间使用日志文件组，就无法删除日志文件组。

+   只要表空间包含任何数据文件，就无法删除表空间。

+   只要仍然有任何使用该表空间的表，就无法从表空间中删除任何数据文件。

+   无法删除与创建文件时使用不同表空间相关联的文件。

例如，要删除本节中到目前为止创建的所有对象，可以使用以下语句：

```sql
mysql> DROP TABLE dt_1;

mysql> ALTER TABLESPACE ts_1
 -> DROP DATAFILE 'data_2.dat'
 -> ENGINE NDBCLUSTER;

mysql> ALTER TABLESPACE ts_1
 -> DROP DATAFILE 'data_1.dat'
 -> ENGINE NDBCLUSTER;

mysql> DROP TABLESPACE ts_1
 -> ENGINE NDBCLUSTER;

mysql> DROP LOGFILE GROUP lg_1
 -> ENGINE NDBCLUSTER;
```

这些语句必须按照显示的顺序执行，除了两个`ALTER TABLESPACE ... DROP DATAFILE`语句可以以任意顺序执行。
