# 28.3.15 INFORMATION_SCHEMA FILES 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-files-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-files-table.html)

`FILES`表提供关于存储 MySQL 表空间数据的文件的信息。

`FILES`表提供关于`InnoDB`数据文件的信息。在 NDB 集群中，此表还提供 NDB 集群磁盘数据表存储的文件信息。有关特定于`InnoDB`的其他信息，请参见本节后面的 InnoDB 注释；有关特定于 NDB 集群的其他信息，请参见 NDB 注释。

`FILES`表具有以下列：

+   `FILE_ID`

    对于`InnoDB`：表空间 ID，也称为`space_id`或`fil_space_t::id`。

    对于`NDB`：文件标识符。`FILE_ID`列值是自动生成的。

+   `FILE_NAME`

    对于`InnoDB`：数据文件的名称。每个表和通用表空间都有一个`.ibd`文件扩展名。撤销表空间以`undo`为前缀。系统表空间以`ibdata`为前缀。全局临时表空间以`ibtmp`为前缀。文件名包括文件路径，可能相对于 MySQL 数据目录（`datadir`系统变量的值）。

    对于`NDB`：由`CREATE LOGFILE GROUP`或`ALTER LOGFILE GROUP`创建的撤销日志文件的名称，或由`CREATE TABLESPACE`或`ALTER TABLESPACE`创建的数据文件的名称。在 NDB 8.0 中，文件名显示为相对路径；对于撤销日志文件，此路径相对于目录``DataDir`/ndb_`NodeId`_fs/LG`；对于数据文件，相对于目录``DataDir`/ndb_`NodeId`_fs/TS`。这意味着，例如，使用`ALTER TABLESPACE ts ADD DATAFILE 'data_2.dat' INITIAL SIZE 256M`创建的数据文件的名称显示为`./data_2.dat`。

+   `FILE_TYPE`

    对于`InnoDB`：表空间文件类型。对于`InnoDB`文件，有三种可能的文件类型。`TABLESPACE`是任何系统、通用或每个表的表空间文件的文件类型，用于保存表、索引或其他形式的用户数据。`TEMPORARY`是临时表空间的文件类型。`UNDO LOG`是撤销表空间的文件类型，用于保存撤销记录。

    对于`NDB`：值之一为`UNDO LOG`或`DATAFILE`。在 NDB 8.0.13 之前，`TABLESPACE`也是可能的值。

+   `TABLESPACE_NAME`

    与文件关联的表空间的名称。

    对于`InnoDB`：一般表空间名称在创建时指定。每表每文件表空间名称显示格式如下：`*`schema_name`*/*`table_name`*`。`InnoDB`系统表空间名称为`innodb_system`。全局临时表空间名称为`innodb_temporary`。默认撤销表空间名称��`inndb_undo_001`和`inndb_undo_002`。用户创建的撤销表空间名称在创建时指定。

+   `TABLE_CATALOG`

    此值始终为空。

+   `TABLE_SCHEMA`

    这总是`NULL`。

+   `TABLE_NAME`

    这总是`NULL`。

+   `LOGFILE_GROUP_NAME`

    对于`InnoDB`：这总是`NULL`。

    对于`NDB`：日志文件或数据文件所属的日志文件组的名称。

+   `LOGFILE_GROUP_NUMBER`

    对于`InnoDB`：这总是`NULL`。

    对于`NDB`：对于磁盘数据撤销日志文件，日志文件所属的日志文件组的自动生成的 ID 号。这与`ndbinfo.dict_obj_info`表中的`id`列和`ndbinfo.logspaces`表中的`log_id`列以及此撤销日志文件的`ndbinfo.logspaces`表中的值相同。

+   `ENGINE`

    对于`InnoDB`：此值始终为`InnoDB`。

    对于`NDB`：此值始终为`ndbcluster`。

+   `FULLTEXT_KEYS`

    这总是`NULL`。

+   `DELETED_ROWS`

    这总是`NULL`。

+   `UPDATE_COUNT`

    这总是`NULL`。

+   `FREE_EXTENTS`

    对于`InnoDB`：当前数据文件中完全空闲的扩展数。

    对于`NDB`：尚未被文件使用的扩展数。

+   `TOTAL_EXTENTS`

    对于`InnoDB`：当前数据文件中使用的完整扩展数。文件末尾的任何部分扩展都不计算在内。

    对于`NDB`：分配给文件的总扩展数。

+   `EXTENT_SIZE`

    对于`InnoDB`：对于具有 4KB、8KB 或 16KB 页面大小的文件，扩展大小为 1048576（1MB）。对于具有 32KB 页面大小的文件，扩展大小为 2097152 字节（2MB），对于具有 64KB 页面大小的文件，扩展大小为 4194304（4MB）。`FILES`不报告`InnoDB`页面大小。页面大小由`innodb_page_size`系统变量定义。扩展大小信息也可以从`INNODB_TABLESPACES`表中检索，其中`FILES.FILE_ID = INNODB_TABLESPACES.SPACE`。

    对于`NDB`：文件的扩展大小（以字节为单位）。

+   `INITIAL_SIZE`

    对于`InnoDB`：文件的初始大小（以字节为单位）。

    对于`NDB`：文件的大小（以字节为单位）。这是在`CREATE LOGFILE GROUP`、`ALTER LOGFILE GROUP`、`CREATE TABLESPACE`或`ALTER TABLESPACE`语句中用于创建文件的`INITIAL_SIZE`子句中使用的相同值。

+   `MAXIMUM_SIZE`

    对于`InnoDB`：文件中允许的最大字节数。对于除了预定义系统表空间数据文件之外的所有数据文件，该值为`NULL`。最大系统表空间文件大小由`innodb_data_file_path`定义。最大全局临时表空间文件大小由`innodb_temp_data_file_path`定义。对于预定义系统表空间数据文件的`NULL`值表示未明确定义文件大小限制。

    对于`NDB`：此值始终与`INITIAL_SIZE`值相同。

+   `AUTOEXTEND_SIZE`

    表空间的自动扩展大小。对于`NDB`，`AUTOEXTEND_SIZE`始终为`NULL`。

+   `CREATION_TIME`

    这始终为`NULL`。

+   `LAST_UPDATE_TIME`

    这始终为`NULL`。

+   `LAST_ACCESS_TIME`

    这始终为`NULL`。

+   `RECOVER_TIME`

    这始终为`NULL`。

+   `TRANSACTION_COUNTER`

    这始终为`NULL`。

+   `VERSION`

    对于`InnoDB`：这始终为`NULL`。

    对于`NDB`：文件的版本号。

+   `ROW_FORMAT`

    对于`InnoDB`：这始终为`NULL`。

    对于`NDB`：`FIXED`或`DYNAMIC`之一。

+   `TABLE_ROWS`

    这始终为`NULL`。

+   `AVG_ROW_LENGTH`

    这始终为`NULL`。

+   `DATA_LENGTH`

    这始终为`NULL`。

+   `MAX_DATA_LENGTH`

    这始终为`NULL`。

+   `INDEX_LENGTH`

    这始终为`NULL`。

+   `DATA_FREE`

    对于`InnoDB`：整个表空间的空闲空间总量（以字节为单位）。预定义系统表空间，包括系统表空间和临时表表空间，可能有一个或多个数据文件。

    对于`NDB`：这始终为`NULL`。

+   `CREATE_TIME`

    这始终为`NULL`。

+   `UPDATE_TIME`

    这始终为`NULL`。

+   `CHECK_TIME`

    这始终为`NULL`。

+   `CHECKSUM`

    这始终为`NULL`。

+   `STATUS`

    对于`InnoDB`：默认情况下，此值为`NORMAL`。`InnoDB`文件表表空间可能报告`IMPORTING`，表示表空间尚不可用。

    对于`NDB`：对于 NDB Cluster Disk Data 文件，此值始终为`NORMAL`。

+   `EXTRA`

    对于`InnoDB`：这始终为`NULL`。

    对于`NDB`：（*NDB 8.0.15 及更高版本*）对于撤销日志文件，此列显示撤销日志缓冲区大小；对于数据文件，始终为*NULL*。下面几段提供了更详细的解释。

    `NDBCLUSTER`在集群中的每个数据节点上存储每个数据文件和每个撤销日志文件的副本。在 NDB 8.0.13 及更高版本中，`FILES`表仅包含每个此类文件的一行。假设您在具有四个数据节点的 NDB 集群上运行以下两个语句：

    ```sql
    CREATE LOGFILE GROUP mygroup
        ADD UNDOFILE 'new_undo.dat'
        INITIAL_SIZE 2G
        ENGINE NDBCLUSTER;

    CREATE TABLESPACE myts
        ADD DATAFILE 'data_1.dat'
        USE LOGFILE GROUP mygroup
        INITIAL_SIZE 256M
        ENGINE NDBCLUSTER;
    ```

    运行这两个语句成功后，您应该看到与针对`FILES`表的查询所示的结果类似的结果：

    ```sql
    mysql> SELECT LOGFILE_GROUP_NAME, FILE_TYPE, EXTRA
     ->     FROM INFORMATION_SCHEMA.FILES
     ->     WHERE ENGINE = 'ndbcluster';

    +--------------------+-----------+--------------------------+
    | LOGFILE_GROUP_NAME | FILE_TYPE | EXTRA                    |
    +--------------------+-----------+--------------------------+
    | mygroup            | UNDO LOG  | UNDO_BUFFER_SIZE=8388608 |
    | mygroup            | DATAFILE  | NULL                     |
    +--------------------+-----------+--------------------------+
    ```

    NDB 8.0.13 中意外删除了撤销日志缓冲区大小信息，但在 NDB 8.0.15 中恢复了。（Bug #92796，Bug #28800252）

    在 NDB 8.0.13 之前，`FILES`表对于每个数据节点上的每个文件都包含一行，以及其撤销缓冲区的大小。在这些版本中，相同查询的结果每个数据节点包含一行，如下所示：

    ```sql
    +--------------------+-----------+-----------------------------------------+
    | LOGFILE_GROUP_NAME | FILE_TYPE | EXTRA                                   |
    +--------------------+-----------+-----------------------------------------+
    | mygroup            | UNDO LOG  | CLUSTER_NODE=5;UNDO_BUFFER_SIZE=8388608 |
    | mygroup            | UNDO LOG  | CLUSTER_NODE=6;UNDO_BUFFER_SIZE=8388608 |
    | mygroup            | UNDO LOG  | CLUSTER_NODE=7;UNDO_BUFFER_SIZE=8388608 |
    | mygroup            | UNDO LOG  | CLUSTER_NODE=8;UNDO_BUFFER_SIZE=8388608 |
    | mygroup            | DATAFILE  | CLUSTER_NODE=5                          |
    | mygroup            | DATAFILE  | CLUSTER_NODE=6                          |
    | mygroup            | DATAFILE  | CLUSTER_NODE=7                          |
    | mygroup            | DATAFILE  | CLUSTER_NODE=8                          |
    +--------------------+-----------+-----------------------------------------+
    ```

#### 注意

+   `FILES`是一个非标准的`INFORMATION_SCHEMA`表。

+   截至 MySQL 8.0.21，您必须具有`PROCESS`权限才能查询此表。

#### InnoDB 注意事项

以下注意事项适用于`InnoDB`数据文件。

+   `FILES`报告的信息是从`InnoDB`内存中的打开文件缓存中获取的，而`INNODB_DATAFILES`从`InnoDB`的`SYS_DATAFILES`内部数据字典表中获取数据。

+   `FILES`提供的信息包括全局临时表空间信息，这些信息在`InnoDB`的`SYS_DATAFILES`内部数据字典表中不可用，因此不包括在`INNODB_DATAFILES`中。

+   当存在单独的撤销表空间时，`FILES`中显示撤销表空间信息，这在 MySQL 8.0 中是默认情况。

+   以下查询返回与`InnoDB`表空间相关的所有`FILES`表信息。

    ```sql
    SELECT
      FILE_ID, FILE_NAME, FILE_TYPE, TABLESPACE_NAME, FREE_EXTENTS,
      TOTAL_EXTENTS, EXTENT_SIZE, INITIAL_SIZE, MAXIMUM_SIZE,
      AUTOEXTEND_SIZE, DATA_FREE, STATUS
    FROM INFORMATION_SCHEMA.FILES
    WHERE ENGINE='InnoDB'\G
    ```

#### NDB 注意事项

+   `FILES`表仅提供有关磁盘数据*文件*的信息；您不能用它来确定单个`NDB`表的磁盘空间分配或可用性。但是，可以使用**ndb_desc**来查看每个存储在磁盘上的`NDB`表分配了多少空间，以及该表在磁盘上存储数据的剩余可用空间。

+   从 NDB 8.0.29 开始，`FILES`表中的许多信息也可以在`ndbinfo.files`表中找到。

+   `CREATION_TIME`、`LAST_UPDATE_TIME`和`LAST_ACCESSED`的值由操作系统报告，而不是由`NDB`存储引擎提供。如果操作系统未提供值，则这些列显示`NULL`。

+   `TOTAL EXTENTS`和`FREE_EXTENTS`列之间的差值是当前文件使用的区段数：

    ```sql
    SELECT TOTAL_EXTENTS - FREE_EXTENTS AS extents_used
        FROM INFORMATION_SCHEMA.FILES
        WHERE FILE_NAME = './myfile.dat';
    ```

    要近似计算文件使用的磁盘空间量，请将该差值乘以`EXTENT_SIZE`列的值，该值表示文件的一个区段的大小（以字节为单位）：

    ```sql
    SELECT (TOTAL_EXTENTS - FREE_EXTENTS) * EXTENT_SIZE AS bytes_used
        FROM INFORMATION_SCHEMA.FILES
        WHERE FILE_NAME = './myfile.dat';
    ```

    同样，可以通过将`FREE_EXTENTS`乘以`EXTENT_SIZE`来估算给定文件中剩余可用空间的量：

    ```sql
    SELECT FREE_EXTENTS * EXTENT_SIZE AS bytes_free
        FROM INFORMATION_SCHEMA.FILES
        WHERE FILE_NAME = './myfile.dat';
    ```

    重要提示

    前述查询产生的字节值仅为近似值，其精度与`EXTENT_SIZE`的值成反比。也就是说，`EXTENT_SIZE`越大，近似值的准确性就越低。

    还要记住，一旦使用了一个区段，就不能再次释放它，除非删除包含它的数据文件。这意味着从磁盘数据表中删除的数据*不会*释放磁盘空间。

    可以在`CREATE TABLESPACE`语句中设置区段大小。有关更多信息，请参见第 15.1.21 节，“CREATE TABLESPACE Statement”。

+   在 NDB 8.0.13 之前，在创建日志文件组后，`FILES`表中存在一个额外的行，`FILE_NAME`列中为`NULL`。在 NDB 8.0.13 及更高版本中，不再显示这一行—该行不对应任何文件，并且需要查询`ndbinfo.logspaces`表以获取撤销日志文件使用信息。有关更多信息，请参见此表的描述以及第 25.6.11.1 节，“NDB Cluster Disk Data Objects”。

    本项目中剩余的讨论仅适用于 NDB 8.0.12 及更早版本。对于`FILE_NAME`列中为`NULL`的行，`FILE_ID`列的值始终为`0`，`FILE_TYPE`列的值始终为`UNDO LOG`，`STATUS`列的值始终为`NORMAL`。`ENGINE`列的值始终为`ndbcluster`。

    在此行中，`FREE_EXTENTS`列显示给定日志文件组的所有撤销文件可用的总空闲区段数，其名称和编号分别显示在`LOGFILE_GROUP_NAME`和`LOGFILE_GROUP_NUMBER`列中。

    假设您的 NDB Cluster 上没有现有的日志文件组，并且使用以下语句创建一个：

    ```sql
    mysql> CREATE LOGFILE GROUP lg1
             ADD UNDOFILE 'undofile.dat'
             INITIAL_SIZE = 16M
             UNDO_BUFFER_SIZE = 1M
             ENGINE = NDB;
    ```

    当您查询 `FILES` 表时，现在可以看到这个`NULL`行：

    ```sql
    mysql> SELECT DISTINCT
             FILE_NAME AS File,
             FREE_EXTENTS AS Free,
             TOTAL_EXTENTS AS Total,
             EXTENT_SIZE AS Size,
             INITIAL_SIZE AS Initial
             FROM INFORMATION_SCHEMA.FILES;
    +--------------+---------+---------+------+----------+
    | File         | Free    | Total   | Size | Initial  |
    +--------------+---------+---------+------+----------+
    | undofile.dat |    NULL | 4194304 |    4 | 16777216 |
    | NULL         | 4184068 |    NULL |    4 |     NULL |
    +--------------+---------+---------+------+----------+
    ```

    用于撤消日志记录的可用自由扩展总数始终略少于日志文件组中所有撤消文件的`TOTAL_EXTENTS`列值之和，这是由于维护撤消文件所需的开销。可以通过向日志文件组添加第二个撤消文件，然后针对 `FILES` 表重复上一个查询来看到这一点：

    ```sql
    mysql> ALTER LOGFILE GROUP lg1
             ADD UNDOFILE 'undofile02.dat'
             INITIAL_SIZE = 4M
             ENGINE = NDB;

    mysql> SELECT DISTINCT
             FILE_NAME AS File,
             FREE_EXTENTS AS Free,
             TOTAL_EXTENTS AS Total,
             EXTENT_SIZE AS Size,
             INITIAL_SIZE AS Initial
             FROM INFORMATION_SCHEMA.FILES;
    +----------------+---------+---------+------+----------+
    | File           | Free    | Total   | Size | Initial  |
    +----------------+---------+---------+------+----------+
    | undofile.dat   |    NULL | 4194304 |    4 | 16777216 |
    | undofile02.dat |    NULL | 1048576 |    4 |  4194304 |
    | NULL           | 5223944 |    NULL |    4 |     NULL |
    +----------------+---------+---------+------+----------+
    ```

    通过将自由扩展的数量乘以初始大小来近似计算使用此日志文件组的磁盘数据表的撤消日志记录的可用空间的字节数：

    ```sql
    mysql> SELECT
             FREE_EXTENTS AS 'Free Extents',
             FREE_EXTENTS * EXTENT_SIZE AS 'Free Bytes'
             FROM INFORMATION_SCHEMA.FILES
             WHERE LOGFILE_GROUP_NAME = 'lg1'
             AND FILE_NAME IS NULL;
    +--------------+------------+
    | Free Extents | Free Bytes |
    +--------------+------------+
    |      5223944 |   20895776 |
    +--------------+------------+
    ```

    如果您创建了一个 NDB 集群磁盘数据表，并插入了一些行，您可以看到之后大约还有多少空间用于撤消日志记录，例如：

    ```sql
    mysql> CREATE TABLESPACE ts1
             ADD DATAFILE 'data1.dat'
             USE LOGFILE GROUP lg1
             INITIAL_SIZE 512M
             ENGINE = NDB;

    mysql> CREATE TABLE dd (
             c1 INT NOT NULL PRIMARY KEY,
             c2 INT,
             c3 DATE
             )
             TABLESPACE ts1 STORAGE DISK
             ENGINE = NDB;

    mysql> INSERT INTO dd VALUES
             (NULL, 1234567890, '2007-02-02'),
             (NULL, 1126789005, '2007-02-03'),
             (NULL, 1357924680, '2007-02-04'),
             (NULL, 1642097531, '2007-02-05');

    mysql> SELECT
             FREE_EXTENTS AS 'Free Extents',
             FREE_EXTENTS * EXTENT_SIZE AS 'Free Bytes'
             FROM INFORMATION_SCHEMA.FILES
             WHERE LOGFILE_GROUP_NAME = 'lg1'
             AND FILE_NAME IS NULL;
    +--------------+------------+
    | Free Extents | Free Bytes |
    +--------------+------------+
    |      5207565 |   20830260 |
    +--------------+------------+
    ```

+   在 NDB 8.0.13 之前，`FILES` 表中为每个 NDB 集群磁盘数据表空间存在额外的行。因为它不对应实际文件，所以在 NDB 8.0.13 中被移除。该行的`FILE_NAME`列值为`NULL`，`FILE_ID`列值始终为`0`，`FILE_TYPE`列值始终为`TABLESPACE`，`STATUS`列值始终为`NORMAL`，`ENGINE`列值始终为`NDBCLUSTER`。

    在 NDB 8.0.13 及更高版本中，您可以使用 **ndb_desc** 实用程序获取有关磁盘数据表空间的信息。有关更多信息，请参阅 第 25.6.11.1 节，“NDB 集群磁盘数据对象” 的描述，以及 **ndb_desc** 的说明。

+   有关更多信息以及有关创建、删除和获取有关 NDB 集群磁盘数据对象的示例，请参阅 第 25.6.11 节，“NDB 集群磁盘数据表”。
