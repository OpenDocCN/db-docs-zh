# 15.1.21 创建表空间语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-tablespace.html`](https://dev.mysql.com/doc/refman/8.0/en/create-tablespace.html)

```sql
CREATE [UNDO] TABLESPACE *tablespace_name*

  *InnoDB and NDB:*
    [ADD DATAFILE '*file_name*']
    [AUTOEXTEND_SIZE [=] *value*]

  *InnoDB only:*
    [FILE_BLOCK_SIZE = value]
    [ENCRYPTION [=] {'Y' | 'N'}]

  *NDB only:*
    USE LOGFILE GROUP *logfile_group*
    [EXTENT_SIZE [=] *extent_size*]
    [INITIAL_SIZE [=] *initial_size*]
    [MAX_SIZE [=] *max_size*]
    [NODEGROUP [=] *nodegroup_id*]
    [WAIT]
    [COMMENT [=] '*string*']

  *InnoDB and NDB:*
    [ENGINE [=] *engine_name*]

  *Reserved for future use:*
    [ENGINE_ATTRIBUTE [=] '*string*']
```

此语句用于创建一个表空间。精确的语法和语义取决于所使用的存储引擎。在标准 MySQL 版本中，这始终是一个`InnoDB`表空间。MySQL NDB Cluster 还支持使用`NDB`存储引擎的表空间。

+   InnoDB 的考虑事项

+   NDB Cluster 的考虑事项

+   选项

+   注意事项

+   InnoDB 示例

+   NDB 示例

#### InnoDB 的考虑事项

`CREATE TABLESPACE`语法用于创建通用表空间或撤销表空间。在 MySQL 8.0.14 中引入的`UNDO`关键字必须指定以创建撤销表空间。

通用表空间是一个共享表空间。它可以容纳多个表，并支持所有表行格式。通用表空间可以相对于数据目录或独立于数据目录创建。

创建`InnoDB`通用表空间后，使用[`CREATE TABLE *`tbl_name`* ... TABLESPACE [=] *`tablespace_name`*`](create-table.html "15.1.20 CREATE TABLE Statement")或[`ALTER TABLE *`tbl_name`* TABLESPACE [=] *`tablespace_name`*`](alter-table.html "15.1.9 ALTER TABLE Statement")来将表添加到表空间中。更多信息，请参见第 17.6.3.3 节，“通用表空间”。

撤销表空间包含撤销日志。可以通过指定完全限定的数据文件路径在所选位置创建撤销表空间。更多信息，请参见第 17.6.3.4 节，“撤销表空间”。

#### NDB Cluster 的考虑事项

此语句用于创建一个表空间，可以包含一个或多个数据文件，为 NDB Cluster Disk Data 表提供存储空间（参见第 25.6.11 节，“NDB Cluster Disk Data Tables”）。使用此语句创建一个数据文件并将其添加到表空间中。可以使用`ALTER TABLESPACE`语句（参见第 15.1.10 节，“ALTER TABLESPACE Statement”）向表空间添加其他数据文件。

注意

所有 NDB 集群磁盘数据对象共享相同的命名空间。这意味着*每个磁盘数据对象*必须具有唯一的名称（而不仅仅是给定类型的每个磁盘数据对象）。例如，您不能拥有具有相同名称的表空间和日志文件组，或者具有相同名称的表空间和数据文件。

一个或多个`UNDO`日志文件的日志文件组必须分配给要使用`USE LOGFILE GROUP`子句创建的表空间。*`logfile_group`*必须是使用`CREATE LOGFILE GROUP`创建的现有日志文件组（参见 Section 15.1.16, “CREATE LOGFILE GROUP Statement”）。多个表空间可以使用相同的日志文件组进行`UNDO`日志记录。

在设置`EXTENT_SIZE`或`INITIAL_SIZE`时，您可以选择在数字后面跟随一个数量级的单字母缩写，类似于`my.cnf`中使用的缩写。通常，这是`M`（表示兆字节）或`G`（表示千兆字节）中的一个字母。

`INITIAL_SIZE`和`EXTENT_SIZE`将按以下方式进行四舍五入：

+   `EXTENT_SIZE`四舍五入到最接近的 32K 的整数倍。

+   `INITIAL_SIZE`向下四舍五入到最接近的 32K 的整数倍；此结果向上四舍五入到最接近`EXTENT_SIZE`的整数倍（在任何四舍五入之后）。

注意

`NDB`为数据节点重新启动操作保留了表空间的 4%。此保留空间不能用于数据存储。

描述的四舍五入是显式完成的，并且当 MySQL 服务器执行任何此类四舍五入时会发出警告。这些四舍五入的值也被 NDB 内核用于计算`INFORMATION_SCHEMA.FILES`列值和其他用途。然而，为了避免意外结果，我们建议您在指定这些选项时始终使用 32K 的整数倍。

当使用`ENGINE [=] NDB`进行`CREATE TABLESPACE`时，在每个集群数据节点上创建一个表空间和关联的数据文件。您可以通过查询信息模式`FILES`表来验证数据文件是否已创建并获取有关它们的信息。（请参见本节后面的示例。）

（请参见 Section 28.3.15, “The INFORMATION_SCHEMA FILES Table”。）

#### 选项

+   `ADD DATAFILE`：定义表空间数据文件的名称。在创建`NDB`表空间时，此选项始终是必需的；对于 MySQL 8.0.14 及更高版本的`InnoDB`，仅在创建撤销表空间时才是必需的。`*`file_name`*`，包括任何指定的路径，必须用单引号或双引号括起来。文件名（不包括文件扩展名）和目录名必须至少为一个字节的长度。不支持零长度的文件名和目录名。

    由于`InnoDB`和`NDB`在处理数据文件方面存在相当大的差异，因此在接下来的讨论中，这两种存储引擎将分别进行讨论。

    **InnoDB 数据文件。** 一个`InnoDB`表空间仅支持单个数据文件，其名称必须包含`.ibd`扩展名。

    要将`InnoDB`通用表空间数据文件放在数据目录之外的位置，包括完全限定的路径或相对于数据目录的路径。仅允许为撤销表空间指定完全限定路径。如果不指定路径，则通用表空间将在数据目录中创建。未指定路径创建的撤销表空间将在由`innodb_undo_directory`变量定义的目录中创建。如果未定义`innodb_undo_directory`变量，则撤销表空间将在数据目录中创建。

    为避免与隐式创建的按表创建文件表空间发生冲突，不支持在数据目录的子目录下创建`InnoDB`通用表空间。在数据目录之外创建通用表空间或撤销表空间时，目录必须存在，并且在创建表空间之前必须为`InnoDB`所知。要使目录为`InnoDB`所知，将其添加到`innodb_directories`值或将其添加到其值附加到`innodb_directories`值的变量之一。`innodb_directories`是一个只读变量。配置它需要重新启动服务器。

    如果在创建`InnoDB`表空间时未指定`ADD DATAFILE`子句，则将隐式创建具有唯一文件名的表空间数据文件。唯一文件名是一个 128 位 UUID，格式为由短横线分隔的五组十六进制数字。如果存储引擎需要，将添加文件扩展名。对于`InnoDB`通用表空间数据文件，将添加`.ibd`文件扩展名。在复制环境中，在复制源服务器上创建的数据文件名与在副本上创建的数据文件名不同。

    截至 MySQL 8.0.17，`ADD DATAFILE`子句在创建`InnoDB`表空间时不允许循环目录引用。例如，以下语句中的循环目录引用（`/../`）是不允许的：

    ```sql
    CREATE TABLESPACE ts1 ADD DATAFILE ts1.ibd '*any_directory*/../ts1.ibd';
    ```

    在 Linux 上存在一个例外，如果前面的目录是一个符号链接，则允许循环目录引用。例如，如果*`any_directory`*是一个符号链接，则上面示例中的数据文件路径是允许的。（数据文件路径仍然可以以'`../`'开头。）

    **NDB 数据文件。** 一个`NDB`表空间支持多个数据文件，这些文件可以具有任何合法的文件名；可以使用`ALTER TABLESPACE`语句在创建后向 NDB 集群表空间添加更多数据文件。

    默认情况下，`NDB`表空间数据文件会在数据节点文件系统目录中创建，即在数据节点的数据目录（`DataDir`）下名为`ndb_*`nodeid`*_fs/TS`的目录中，其中*`nodeid`*是数据节点的`NodeId`。要将数据文件放在除默认位置之外的位置，请包含绝对目录路径或相对于默认位置的路径。如果指定的目录不存在，`NDB`会尝试创建它；数据节点进程正在运行的系统用户帐户必须具有适当的权限才能这样做。

    注意

    在确定数据文件路径时，`NDB`不会展开`~`（波浪号）字符。

    当在同一物理主机上运行多个数据节点时，以下注意事项适用：

    +   创建数据文件时不能指定绝对路径。

    +   除非每个数据节点都有单独的数据目录，否则无法在数据节点文件系统目录之外创建表空间数据文件。

    +   如果每个数据节点都有自己的数据目录，则数据文件可以在该目录的任何位置创建。

    +   如果每个数据节点都有自己的数据目录，那么也可以通过相对路径在节点的数据目录之外创建数据文件，只要这个路径对于在主机文件系统上运行的每个数据节点来说都解析为唯一位置。

+   `FILE_BLOCK_SIZE`：此选项专门针对`InnoDB`通用表空间，对于`NDB`是被忽略的，它定义了表空间数据文件的块大小。值可以用字节或千字节来指定。例如，可以将 8 千字节的文件块大小指定为 8192 或 8K。如果不指定此选项，`FILE_BLOCK_SIZE`默认为`innodb_page_size`的值。当您打算将表空间用于存储压缩的`InnoDB`表（`ROW_FORMAT=COMPRESSED`）时，必须定义表空间的`FILE_BLOCK_SIZE`。

    如果`FILE_BLOCK_SIZE`等于`innodb_page_size`值，则表空间只能包含具有未压缩行格式（`COMPACT`，`REDUNDANT`和`DYNAMIC`）的表。具有`COMPRESSED`行格式的表具有与未压缩表不同的物理页大小。因此，压缩表不能与未压缩表共存于同一表空间中。

    对于包含压缩表的通用表空间，必须指定`FILE_BLOCK_SIZE`，并且`FILE_BLOCK_SIZE`的值必须是与`innodb_page_size`值相关的有效压缩页大小。此外，压缩表的物理页大小（`KEY_BLOCK_SIZE`）必须等于`FILE_BLOCK_SIZE/1024`。例如，如果`innodb_page_size=16K`，而`FILE_BLOCK_SIZE=8K`，则表的`KEY_BLOCK_SIZE`必须为 8。更多信息，请参见 Section 17.6.3.3, “通用表空间”。

+   `USE LOGFILE GROUP`：对于`NDB`必需，这是先前使用`CREATE LOGFILE GROUP`创建的日志文件组的名称。对于`InnoDB`不支持，会导致错误。

+   `EXTENT_SIZE`：此选项特定于 NDB，在 InnoDB 中不受支持，会导致错误。`EXTENT_SIZE`设置表空间中任何文件使用的 extent 的大小（以字节为单位）。默认值为 1M。最小大小为 32K，理论上的最大值为 2G，尽管实际最大大小取决于许多因素。在大多数情况下，更改 extent 大小对性能没有任何可测量的影响，建议除了最不寻常的情况外，使用默认值。

    一个 extent 是磁盘空间分配的单位。一个 extent 填满尽可能多的数据，然后使用另一个 extent。理论上，每个数据文件最多可以使用 65,535（64K）个 extent；然而，建议的最大值为 32,768（32K）。单个数据文件的建议最大大小为 32G，即 32K extents × 每个 extent 1MB。此外，一旦将 extent 分配给给定分区，就不能用于存储来自不同分区的数据；一个 extent 不能存储来自多个分区的数据。这意味着，例如，一个具有单个数据文件的表空间，其`INITIAL_SIZE`（在下一项中描述）为 256 MB，`EXTENT_SIZE`为 128M，只有两个 extent，因此最多可以用于存储来自最多两个不同磁盘数据表分区的数据。

    通过查询信息模式`FILES`表，您可以看到给定数据文件中剩余的空间有多少，从而估算文件中剩余的空间量。有关进一步讨论和示例，请参见 Section 28.3.15, “The INFORMATION_SCHEMA FILES Table”。

+   `INITIAL_SIZE`：此选项特定于`NDB`，不受`InnoDB`支持，在那里会出现错误。

    `INITIAL_SIZE`参数设置了使用`ADD DATATFILE`指定的数据文件的总大小（以字节为单位）。创建了此文件后，其大小不能更改；但是，您可以使用`ALTER TABLESPACE ... ADD DATAFILE`向表空间添加更多数据文件。

    `INITIAL_SIZE`是可选的；其默认值为 134217728（128 MB）。

    在 32 位系统上，`INITIAL_SIZE`的最大支持值为 4294967296（4 GB）。

+   `AUTOEXTEND_SIZE`：在 MySQL 8.0.23 之前被 MySQL 忽略；从 MySQL 8.0.23 开始，定义了`InnoDB`在表空间变满时扩展的量。设置必须是 4MB 的倍数。默认设置为 0，这将导致表空间根据隐式默认行为进行扩展。有关更多信息，请参见 Section 17.6.3.9, “Tablespace AUTOEXTEND_SIZE Configuration”。

    在任何 MySQL NDB Cluster 8.0 的版本中都没有任何影响，无论使用的存储引擎是什么。

+   `MAX_SIZE`：当前 MySQL 忽略；保留以备可能的将来使用。在 MySQL 8.0 或 MySQL NDB Cluster 8.0 的任何版本中都没有任何影响，无论使用的存储引擎是什么。

+   `NODEGROUP`：当前 MySQL 忽略；保留以备可能的将来使用。在 MySQL 8.0 或 MySQL NDB Cluster 8.0 的任何版本中都没有任何影响，无论使用的存储引擎是什么。

+   `WAIT`：当前 MySQL 忽略；保留以备可能的将来使用。在 MySQL 8.0 或 MySQL NDB Cluster 8.0 的任何版本中都没有任何影响，无论使用的存储引擎是什么。

+   `COMMENT`：当前 MySQL 忽略；保留以备可能的将来使用。在 MySQL 8.0 或 MySQL NDB Cluster 8.0 的任何版本中都没有任何影响，无论使用的存储引擎是什么。

+   `ENCRYPTION`子句为`InnoDB`通用表空间启用或禁用页面级数据加密。MySQL 8.0.13 引入了对通用表空间的加密支持。

    截至 MySQL 8.0.16，如果未指定 `ENCRYPTION` 子句，则`default_table_encryption` 设置控制是否启用加密。`ENCRYPTION` 子句会覆盖`default_table_encryption` 设置。但是，如果启用了`table_encryption_privilege_check` 变量，则需要`TABLE_ENCRYPTION_ADMIN` 权限才能使用与`default_table_encryption` 设置不同的 `ENCRYPTION` 子句设置。

    必须在创建启用加密的表空间之前安装和配置一个密钥环插件。

    当通用表空间加密时，驻留在表空间中的所有表都会被加密。同样，创建在加密表空间中的表也会被加密。

    更多信息，请参见 第 17.13 节，“InnoDB 数据静态加密”

+   `ENGINE`：定义使用表空间的存储引擎，其中 *`engine_name`* 是存储引擎的名称。目前，标准 MySQL 8.0 发行版仅支持 `InnoDB` 存储引擎。MySQL NDB Cluster 支持 `NDB` 和 `InnoDB` 表空间。如果未指定选项，则 `ENGINE` 使用 `default_storage_engine` 系统变量的值。

+   `ENGINE_ATTRIBUTE` 选项（自 MySQL 8.0.21 起可用）用于指定主存储引擎的表空间属性。该选项保留供将来使用。

    允许的值是包含有效`JSON`文档的字符串文字或空字符串（''）。无效的`JSON`会被拒绝。

    ```sql
    CREATE TABLESPACE ts1 ENGINE_ATTRIBUTE='{"*key*":"*value*"}';
    ```

    `ENGINE_ATTRIBUTE` 值可以重复而不会出错。在这种情况下，将使用最后指定的值。

    `ENGINE_ATTRIBUTE` 值不会被服务器检查，也不会在表的存储引擎更改时清除。

#### 注意

+   有关 MySQL 表空间命名规则，请参见 第 11.2 节，“模式对象名称”。除了这些规则外，斜杠字符（“/”）不允许使用，也不能使用以 `innodb_` 开头的名称，因为此前缀保留供系统使用。

+   创建临时通用表空间不受支持。

+   通用表空间不支持临时表。

+   `TABLESPACE`选项可用于将`InnoDB`表分区或子分区分配给文件-每表表空间，可与`CREATE TABLE`或`ALTER TABLE`一起使用。所有分区必须属于相同的存储引擎。不支持将表分区分配给共享的 InnoDB 表空间。共享表空间包括 InnoDB 系统表空间和通用表空间。

+   通用表空间支持使用`CREATE TABLE ... TABLESPACE`添加任何行格式的表。不需要启用`innodb_file_per_table`。

+   `innodb_strict_mode`不适用于通用表空间。表空间管理规则严格执行，与`innodb_strict_mode`无关。如果`CREATE TABLESPACE`参数不正确或不兼容，则无论`innodb_strict_mode`设置如何，操作都会失败。当使用`CREATE TABLE ... TABLESPACE`或`ALTER TABLE ... TABLESPACE`将表添加到通用表空间时，将忽略`innodb_strict_mode`，但该语句将被评估为启用了`innodb_strict_mode`。

+   使用`DROP TABLESPACE`来删除表空间。在删除表空间之前，必须使用`DROP TABLE`删除表空间中的所有表。在删除 NDB Cluster 表空间之前，还必须使用一个或多个`ALTER TABLESPACE ... DROP DATATFILE`语句删除所有数据文件。参见 Section 25.6.11.1, “NDB Cluster Disk Data Objects”。

+   添加到 InnoDB 通用表空间的 InnoDB 表的所有部分都驻留在通用表空间中，包括索引和`BLOB`页面。

    对于分配给表空间的 NDB 表，只有未建立索引的列存储在磁盘上，并实际使用表空间数据文件。所有 NDB 表的索引和已建立索引的列始终保留在内存中。

+   与系统表空间类似，截断或删除存储在通用表空间中的表会在通用表空间的.ibd 数据文件中创建内部的可用空间，该空间只能用于新的 InnoDB 数据。与文件-每表表空间不同，空间不会像对操作系统释放那样被释放。

+   通用表空间不与任何数据库或模式相关联。

+   `ALTER TABLE ... DISCARD TABLESPACE` 和 `ALTER TABLE ...IMPORT TABLESPACE` 不支持属于通用表空间的表。

+   服务器对引用通用表空间的 DDL 使用表空间级元数据锁定。相比之下，服务器对引用每个表的文件表空间的 DDL 使用表级元数据锁定。

+   生成的或现有的表空间不能更改为通用表空间。

+   通用表空间名称与每个表的文件表空间名称之间没有冲突。通用表空间名称中不允许出现文件表空间名称中的“/”字符。

+   **mysqldump** 和 **mysqlpump** 不会转储`InnoDB` `CREATE TABLESPACE`语句。

#### InnoDB 示例

此示例演示了创建一个通用表空间，并添加三个不同行格式的未压缩表。

```sql
mysql> CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' ENGINE=INNODB;

mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY) TABLESPACE ts1 ROW_FORMAT=REDUNDANT;

mysql> CREATE TABLE t2 (c1 INT PRIMARY KEY) TABLESPACE ts1 ROW_FORMAT=COMPACT;

mysql> CREATE TABLE t3 (c1 INT PRIMARY KEY) TABLESPACE ts1 ROW_FORMAT=DYNAMIC;
```

此示例演示了创建一个通用表空间并添加一个压缩表。该示例假定默认的`innodb_page_size`值为 16K。8192 的`FILE_BLOCK_SIZE`要求压缩表具有 8 的`KEY_BLOCK_SIZE`。

```sql
mysql> CREATE TABLESPACE `ts2` ADD DATAFILE 'ts2.ibd' FILE_BLOCK_SIZE = 8192 Engine=InnoDB;

mysql> CREATE TABLE t4 (c1 INT PRIMARY KEY) TABLESPACE ts2 ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8;
```

此示例演示了创建一个通用表空间而不指定`ADD DATAFILE`子句，这在 MySQL 8.0.14 中是可选的。

```sql
mysql> CREATE TABLESPACE `ts3` ENGINE=INNODB;
```

此示例演示了创建撤销表空间。

```sql
mysql> CREATE UNDO TABLESPACE *undo_003* ADD DATAFILE '*undo_003*.ibu';
```

#### NDB 示例

假设您希望使用名为`mydata-1.dat`的数据文件创建一个名为`myts`的 NDB Cluster 磁盘数据表空间。一个`NDB`表空间总是需要使用一个或多个撤销日志文件组的日志文件。对于此示例，我们首先创建一个名为`mylg`的日志文件组，其中包含一个名为`myundo-1.dat`的撤销长文件，使用此处显示的`CREATE LOGFILE GROUP`语句：

```sql
mysql> CREATE LOGFILE GROUP myg1
 ->     ADD UNDOFILE 'myundo-1.dat'
 ->     ENGINE=NDB;
Query OK, 0 rows affected (3.29 sec)
```

现在，您可以使用以下语句创建先前描述的表空间：

```sql
mysql> CREATE TABLESPACE myts
 ->     ADD DATAFILE 'mydata-1.dat'
 ->     USE LOGFILE GROUP mylg
 ->     ENGINE=NDB;
Query OK, 0 rows affected (2.98 sec)
```

您现在可以使用带有`TABLESPACE`和`STORAGE DISK`选项的`CREATE TABLE`语句来创建一个磁盘数据表，类似于下面显示的内容：

```sql
mysql> CREATE TABLE mytable (
 ->     id INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
 ->     lname VARCHAR(50) NOT NULL,
 ->     fname VARCHAR(50) NOT NULL,
 ->     dob DATE NOT NULL,
 ->     joined DATE NOT NULL,
 ->     INDEX(last_name, first_name)
 -> )
 ->     TABLESPACE myts STORAGE DISK
 ->     ENGINE=NDB;
Query OK, 0 rows affected (1.41 sec)
```

需要注意的是，由于`id`、`lname`和`fname`列都被索引，因此`mytable`中只有`dob`和`joined`列实际上存储在磁盘上。

如前所述，当`CREATE TABLESPACE`与`ENGINE [=] NDB`一起使用时，将在每个 NDB Cluster 数据节点上创建一个表空间和相关的数据文件。您可以通过查询信息模式`FILES`表来验证数据文件是否已创建并获取有关它们的信息，如下所示：

```sql
mysql> SELECT FILE_NAME, FILE_TYPE, LOGFILE_GROUP_NAME, STATUS, EXTRA
 ->     FROM INFORMATION_SCHEMA.FILES
 ->     WHERE TABLESPACE_NAME = 'myts';

+--------------+------------+--------------------+--------+----------------+
| file_name    | file_type  | logfile_group_name | status | extra          |
+--------------+------------+--------------------+--------+----------------+
| mydata-1.dat | DATAFILE   | mylg               | NORMAL | CLUSTER_NODE=5 |
| mydata-1.dat | DATAFILE   | mylg               | NORMAL | CLUSTER_NODE=6 |
| NULL         | TABLESPACE | mylg               | NORMAL | NULL           |
+--------------+------------+--------------------+--------+----------------+
3 rows in set (0.01 sec)
```

有关更多信息和示例，请参见第 25.6.11.1 节，“NDB 集群磁盘数据对象”。
