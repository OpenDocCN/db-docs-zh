> 原文：[`dev.mysql.com/doc/refman/8.0/en/general-tablespaces.html`](https://dev.mysql.com/doc/refman/8.0/en/general-tablespaces.html)

#### 17.6.3.3 通用表空间

通用表空间是使用`CREATE TABLESPACE`语法创建的共享`InnoDB`表空间。通用表空间的功能和特性在本节中的以下主题下描述：

+   通用表空间功能

+   创建通用表空间

+   向通用表空间添加表

+   通用表空间行格式支持

+   使用 ALTER TABLE 在表空间之间移动表

+   重命名通用表空间

+   删除通用表空间

+   通用表空间限制

##### 通用表空间功能

通用表空间提供以下功能：

+   与系统表空间类似，通用表空间是可以存储多个表数据的共享表空间。

+   通用表空间相对于每表一个表空间具有潜在的内存优势。服务器在表空间的生命周期内将表空间元数据保存在内存中。较少的通用表空间中的多个表消耗的表空间元数据内存比相同数量的表在单独的每表一个表空间中少。

+   通用表空间数据文件可以放置在相对于 MySQL 数据目录或独立于 MySQL 数据目录的目录中，这为您提供了许多与每表一个表空间的数据文件和存储管理功能。与每表一个表空间一样，将数据文件放置在 MySQL 数据目录之外的能力允许您单独管理关键表的性能，为特定表设置 RAID 或 DRBD，或将表绑定到特定磁盘，例如。

+   通用表空间支持所有表行格式和相关功能。

+   `TABLESPACE`选项可与`CREATE TABLE`一起使用，以在通用表空间、每表一个表空间或系统表空间中创建表。

+   `TABLESPACE` 选项可与 `ALTER TABLE` 一起使用，以在通用表空间、每表一个文件的表空间和系统表空间之间移动表。

##### 创建通用表空间

使用 `CREATE TABLESPACE` 语法创建通用表空间。

```sql
CREATE TABLESPACE *tablespace_name*
    [ADD DATAFILE '*file_name*']
    [FILE_BLOCK_SIZE = *value*]
        [ENGINE [=] *engine_name*]
```

通用表空间可以在数据目录内或外创建。为避免与隐式创建的每表一个文件的表空间冲突，不支持在数据目录下的子目录中创建通用表空间。在数据目录之外创建通用表空间时，目录必须存在，并且在创建表空间之前必须为 `InnoDB` 所知。要使未知目录为 `InnoDB` 所知，将目录添加到 `innodb_directories` 参数值中。`innodb_directories` 是一个只读启动选项。配置它需要重新启动服务器。

示例：

在数据目录中创建一个通用表空间：

```sql
mysql> CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' Engine=InnoDB;
```

或

```sql
mysql> CREATE TABLESPACE `ts1` Engine=InnoDB;
```

在 MySQL 8.0.14 中，`ADD DATAFILE` 子句是可选的，在此之前是必需的。如果在创建表空间时未指定 `ADD DATAFILE` 子句，则会隐式创建一个具有唯一文件名的表空间数据文件。唯一文件名是一个 128 位 UUID，格式为五组十六进制数字，用破折号分隔 (*`aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee`*)。通用表空间数据文件包括一个 `.ibd` 文件扩展名。在复制环境中，源上创建的数据文件名与副本上创建的数据文件名不同。

在数据目录之外的目录中创建一个通用表空间：

```sql
mysql> CREATE TABLESPACE `ts1` ADD DATAFILE '/my/tablespace/directory/ts1.ibd' Engine=InnoDB;
```

您可以指定相对于数据目录的路径，只要表空间目录不在数据目录下即可。在此示例中，`my_tablespace` 目录与数据目录处于同一级别：

```sql
mysql> CREATE TABLESPACE `ts1` ADD DATAFILE '../my_tablespace/ts1.ibd' Engine=InnoDB;
```

注意

`ENGINE = InnoDB` 子句必须作为 `CREATE TABLESPACE` 语句的一部分定义，或者 `InnoDB` 必须被定义为默认存储引擎 (`default_storage_engine=InnoDB`)。

##### 将表添加到通用表空间

创建通用表空间后，可以使用 [`CREATE TABLE *`tbl_name`* ... TABLESPACE [=] *`tablespace_name`*`](create-table.html "15.1.20 CREATE TABLE Statement") 或 [`ALTER TABLE *`tbl_name`* TABLESPACE [=] *`tablespace_name`*`](alter-table.html "15.1.9 ALTER TABLE Statement") 语句将表添加到表空间中，如下例所示：

`CREATE TABLE`:

```sql
mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY) TABLESPACE ts1;
```

`ALTER TABLE`:

```sql
mysql> ALTER TABLE t2 TABLESPACE ts1;
```

注意

在 MySQL 5.7.24 中弃用了向共享表空间添加表分区的支持，并在 MySQL 8.0.13 中移除。共享表空间包括`InnoDB`系统表空间和通用表空间。

有关详细的语法信息，请参阅`CREATE TABLE`和`ALTER TABLE`。

##### 通用表空间行格式支持

通用表空间支持所有表行格式（`REDUNDANT`，`COMPACT`，`DYNAMIC`，`COMPRESSED`），但由于不同的物理页大小，压缩表和非压缩表不能共存于同一通用表空间中。

要使通用表空间包含压缩表（`ROW_FORMAT=COMPRESSED`），必须指定`FILE_BLOCK_SIZE`选项，并且`FILE_BLOCK_SIZE`值必须是相对于`innodb_page_size`值的有效压缩页大小。此外，压缩表的物理页大小（`KEY_BLOCK_SIZE`）必须等于`FILE_BLOCK_SIZE/1024`。例如，如果`innodb_page_size=16KB`和`FILE_BLOCK_SIZE=8K`，则表的`KEY_BLOCK_SIZE`必须为 8。

以下表显示了允许的`innodb_page_size`、`FILE_BLOCK_SIZE`和`KEY_BLOCK_SIZE`组合。`FILE_BLOCK_SIZE`的值也可以用字节指定。要确定给定`FILE_BLOCK_SIZE`的有效`KEY_BLOCK_SIZE`值，将`FILE_BLOCK_SIZE`值除以 1024。32K 和 64K 的`InnoDB`页大小不支持表压缩。有关`KEY_BLOCK_SIZE`的更多信息，请参阅`CREATE TABLE`和第 17.9.1.2 节，“创建压缩表”。

**表 17.3 压缩表允许的页大小、FILE_BLOCK_SIZE 和 KEY_BLOCK_SIZE 组合**

| InnoDB 页大小（innodb_page_size） | 允许的 FILE_BLOCK_SIZE 值 | 允许的 KEY_BLOCK_SIZE 值 |
| --- | --- | --- |
| 64KB | 64K (65536) | 不支持压缩 |
| 32KB | 32K (32768) | 不支持压缩 |
| 16KB | 16K (16384) | 无。如果`innodb_page_size`等于`FILE_BLOCK_SIZE`，则表空间不能包含压缩表。 |
| 16KB | 8K (8192) | 8 |
| 16KB | 4K (4096) | 4 |
| 16KB | 2K (2048) | 2 |
| 16KB | 1K (1024) | 1 |
| 8KB | 8K (8192) | 无。如果`innodb_page_size`等于`FILE_BLOCK_SIZE`，则表空间不能包含压缩表。 |
| 8KB | 4K (4096) | 4 |
| 8KB | 2K (2048) | 2 |
| 8KB | 1K (1024) | 1 |
| 4KB | 4K (4096) | 无。如果`innodb_page_size`等于`FILE_BLOCK_SIZE`，则表空间不能包含压缩表。 |
| 4KB | 2K (2048) | 2 |
| 4KB | 1K (1024) | 1 |
| InnoDB 页面大小（innodb_page_size） | 允许的 FILE_BLOCK_SIZE 值 | 允许的 KEY_BLOCK_SIZE 值 |

此示例演示了创建通用表空间并添加压缩表。该示例假定默认的`innodb_page_size`为 16KB。`FILE_BLOCK_SIZE`为 8192 要求压缩表具有 8 的`KEY_BLOCK_SIZE`。

```sql
mysql> CREATE TABLESPACE `ts2` ADD DATAFILE 'ts2.ibd' FILE_BLOCK_SIZE = 8192 Engine=InnoDB;

mysql> CREATE TABLE t4 (c1 INT PRIMARY KEY) TABLESPACE ts2 ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8;
```

在创建通用表空间时如果不指定`FILE_BLOCK_SIZE`，`FILE_BLOCK_SIZE`默认为`innodb_page_size`。当`FILE_BLOCK_SIZE`等于`innodb_page_size`时，表空间只能包含具有未压缩行格式（`COMPACT`，`REDUNDANT`和`DYNAMIC`行格式）的表。

##### 使用 ALTER TABLE 在表空间之间移动表

使用带有`TABLESPACE`选项的`ALTER TABLE`可以将表移动到现有的通用表空间，新的每个表的文件表空间，或系统表空间。

注意

在 MySQL 5.7.24 中，支持将表分区放置在共享表空间中的功能已被弃用，并在 MySQL 8.0.13 中移除。共享表空间包括`InnoDB`系统表空间和通用表空间。

要将表从每个表的文件表空间或系统表空间移动到通用表空间，请指定通用表空间的名称。通用表空间必须存在。有关更多信息，请参阅`ALTER TABLESPACE`。

```sql
ALTER TABLE tbl_name TABLESPACE [=] *tablespace_name*;
```

要将表从通用表空间或每个表的文件表空间移动到系统表空间，请将`innodb_system`指定为表空间名称。

```sql
ALTER TABLE tbl_name TABLESPACE [=] innodb_system;
```

要将表从系统表空间或通用表空间移动到每个表的文件表空间，请将`innodb_file_per_table`指定为表空间名称。

```sql
ALTER TABLE tbl_name TABLESPACE [=] innodb_file_per_table;
```

`ALTER TABLE ... TABLESPACE`操作会导致完整的表重建，即使`TABLESPACE`属性与其先前值相同也是如此。

`ALTER TABLE ... TABLESPACE`语法不支持将表从临时表空间移动到持久表空间。

`DATA DIRECTORY`子句允许与`CREATE TABLE ... TABLESPACE=innodb_file_per_table`结合使用，但否则不支持与`TABLESPACE`选项结合使用。截至 MySQL 8.0.21，`DATA DIRECTORY`子句中指定的目录必须为`InnoDB`所知。有关更多信息，请参阅使用 DATA DIRECTORY 子句。

在从加密表空间移动表时会有一些限制。请参阅加密限制。

##### 重命名通用表空间

支持使用`ALTER TABLESPACE ... RENAME TO`语法重命名通用表空间。

```sql
ALTER TABLESPACE s1 RENAME TO s2;
```

重命名通用表空间需要 `CREATE TABLESPACE` 权限。

无论 `autocommit` 设置如何，`RENAME TO` 操作都会隐式在 `autocommit` 模式下执行。

在对驻留在表空间中的表执行 `LOCK TABLES` 或 `FLUSH TABLES WITH READ LOCK` 时，无法执行 `RENAME TO` 操作。

在重命名表空间时，对通用表空间中的表采取排他性 元数据锁，防止并发的 DDL。支持并发的 DML。

##### 删除通用表空间

`DROP TABLESPACE` 语句用于删除一个 `InnoDB` 通用表空间。

在进行 `DROP TABLESPACE` 操作之前，必须从表空间中删除所有表。如果表空间不为空，`DROP TABLESPACE` 将返回错误。

使用类似以下查询来识别通用表空间中的表。

```sql
mysql> SELECT a.NAME AS space_name, b.NAME AS table_name FROM INFORMATION_SCHEMA.INNODB_TABLESPACES a,
       INFORMATION_SCHEMA.INNODB_TABLES b WHERE a.SPACE=b.SPACE AND a.NAME LIKE 'ts1';
+------------+------------+
| space_name | table_name |
+------------+------------+
| ts1        | test/t1    |
| ts1        | test/t2    |
| ts1        | test/t3    |
+------------+------------+
```

当表空间中的最后一个表被删除时，通用 `InnoDB` 表空间不会自动删除。必须使用 `DROP TABLESPACE *`tablespace_name`*` 明确删除表空间。

通用表空间不属于任何特定数据库。`DROP DATABASE` 操作可以删除属于通用表空间的表，但无法删除表空间，即使 `DROP DATABASE` 操作删除了属于表空间的所有表。

与系统表空间类似，截断或删除存储在通用表空间中的表会在通用表空间的 .ibd 数据文件 中内部创建可用于新的 `InnoDB` 数据的空间。与在 `DROP TABLE` 操作期间删除文件-每表表空间时释放空间到操作系统不同。

此示例演示了如何删除一个 `InnoDB` 通用表空间。通用表空间 `ts1` 创建了一个单表。在删除表空间之前必须先删除表。

```sql
mysql> CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' Engine=InnoDB;

mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY) TABLESPACE ts1 Engine=InnoDB;

mysql> DROP TABLE t1;

mysql> DROP TABLESPACE ts1;
```

注意

`*`tablespace_name`*` 在 MySQL 中是区分大小写的标识符。

##### 通用表空间限制

+   生成的或现有的表空间不能更改为通用表空间。

+   临时通用表空间的创建不受支持。

+   通用表空间不支持临时表。

+   与系统表空间类似，截断或删除存储在通用表空间中的表会在通用表空间的.ibd 数据文件内部创建可用于新的`InnoDB`数据的空闲空间。空间不会像 file-per-table 表空间那样释放回操作系统。

    此外，在共享表空间（通用表空间或系统表空间）中存在的表上进行表复制的`ALTER TABLE`操作可能会增加表空间使用的空间量。这些操作需要与表中的数据和索引一样多的额外空间。表复制的`ALTER TABLE`操作所需的额外空间不会像 file-per-table 表空间那样释放回操作系统。

+   不支持对属于通用表空间的表执行`ALTER TABLE ... DISCARD TABLESPACE`和`ALTER TABLE ...IMPORT TABLESPACE`操作。

+   在 MySQL 5.7.24 中弃用了将表分区放置在通用表空间中的支持，并在 MySQL 8.0.13 中将其移除。

+   在源和副本位于同一主机上的复制环境中，不支持`ADD DATAFILE`子句，因为这会导致源和副本在相同位置创建同名的表空间，这是不支持的。但是，如果省略`ADD DATAFILE`子句，则表空间将在数据目录中以唯一的生成文件名创建，这是允许的。

+   截至 MySQL 8.0.21，通用表空间不能在 undo 表空间目录（`innodb_undo_directory`）中创建，除非该目录被`InnoDB`所知。已知目录是由`datadir`、`innodb_data_home_dir`和`innodb_directories`变量定义的目录。
