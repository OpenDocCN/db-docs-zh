> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-compression-usage.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-compression-usage.html)

#### 17.9.1.2 创建压缩表

可以在 file-per-table 表空间或通用表空间中创建压缩表。表压缩不适用于 InnoDB 系统表空间。系统表空间（空间 0，.ibdata 文件）可以包含用户创建的表，但也包含内部系统数据，这些数据永远不会被压缩。因此，压缩仅适用于存储在文件表表空间或通用表空间中的表（和索引）。

##### 在文件表表空间中创建压缩表

要在文件表表空间中创建压缩表，必须启用`innodb_file_per_table`（默认情况下）。您可以在 MySQL 配置文件（`my.cnf`或`my.ini`）中设置此参数，也可以使用`SET`语句动态设置。

配置`innodb_file_per_table`选项后，在`CREATE TABLE`或`ALTER TABLE`语句中指定`ROW_FORMAT=COMPRESSED`子句或`KEY_BLOCK_SIZE`子句，或两者都指定，以在文件表表空间中创建一个压缩表。

例如，您可以使用以下语句：

```sql
SET GLOBAL innodb_file_per_table=1;
CREATE TABLE t1
 (c1 INT PRIMARY KEY)
 ROW_FORMAT=COMPRESSED
 KEY_BLOCK_SIZE=8;
```

##### 在通用表空间中创建压缩表

要在通用表空间中创建压缩表，必须为通用表空间定义`FILE_BLOCK_SIZE`，这在创建表空间时指定。`FILE_BLOCK_SIZE`值必须是相对于`innodb_page_size`值的有效压缩页大小，并且由`CREATE TABLE`或`ALTER TABLE`的`KEY_BLOCK_SIZE`子句定义的压缩表的页大小必须等于`FILE_BLOCK_SIZE/1024`。例如，如果`innodb_page_size=16384`且`FILE_BLOCK_SIZE=8192`，则表的`KEY_BLOCK_SIZE`必须为 8。有关更多信息，请参见第 17.6.3.3 节，“通用表空间”。

以下示例演示了创建通用表空间并添加压缩表。该示例假定默认的`innodb_page_size`为 16K。`FILE_BLOCK_SIZE`为 8192 要求压缩表具有`KEY_BLOCK_SIZE`为 8。

```sql
mysql> CREATE TABLESPACE `ts2` ADD DATAFILE 'ts2.ibd' FILE_BLOCK_SIZE = 8192 Engine=InnoDB;

mysql> CREATE TABLE t4 (c1 INT PRIMARY KEY) TABLESPACE ts2 ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8;
```

##### 注意

+   截至 MySQL 8.0，压缩表的表空间文件是使用物理页大小而不是`InnoDB`页大小创建的，这使得空压缩表的初始表空间文件大小比以前的 MySQL 版本更小。

+   如果指定`ROW_FORMAT=COMPRESSED`，可以省略`KEY_BLOCK_SIZE`；`KEY_BLOCK_SIZE`设置默认为`innodb_page_size`值的一半。

+   如果指定了有效的`KEY_BLOCK_SIZE`值，可以省略`ROW_FORMAT=COMPRESSED`；压缩会自动启用。

+   要确定`KEY_BLOCK_SIZE`的最佳值，通常需要创建几个具有不同此子句值的相同表的副本，然后测量生成的`.ibd`文件的大小，并查看每个在实际工作负载中的表现如何。对于一般表空间，请记住删除表不会减小一般表空间`.ibd`文件的大小，也不会将磁盘空间返回给操作系统。有关更多信息，请参见第 17.6.3.3 节，“一般表空间”。

+   `KEY_BLOCK_SIZE`值被视为提示；如果需要，`InnoDB`可以使用不同的大小。对于每个表的文件表空间，`KEY_BLOCK_SIZE`只能小于或等于`innodb_page_size`值。如果指定的值大于`innodb_page_size`值，则指定的值将被忽略，发出警告，并将`KEY_BLOCK_SIZE`设置为`innodb_page_size`值的一半。如果`innodb_strict_mode=ON`，指定无效的`KEY_BLOCK_SIZE`值会返回错误。对于一般表空间，有效的`KEY_BLOCK_SIZE`值取决于表空间的`FILE_BLOCK_SIZE`设置。有关更多信息，请参见第 17.6.3.3 节，“一般表空间”。

+   `InnoDB`支持 32KB 和 64KB 页大小，但这些页大小不支持压缩。有关更多信息，请参阅`innodb_page_size`文档。

+   `InnoDB`数据页的默认未压缩大小为 16KB。根据选项值的组合，MySQL 使用 1KB、2KB、4KB、8KB 或 16KB 的页大小用于表空间数据文件（`.ibd`文件）。实际的压缩算法不受`KEY_BLOCK_SIZE`值的影响；该值确定每个压缩块的大小，进而影响可以打包到每个压缩页中的行数。

+   在文件表表空间中创建压缩表时，将 `KEY_BLOCK_SIZE` 设置为 `InnoDB` 页大小 通常不会产生太多压缩效果。例如，设置 `KEY_BLOCK_SIZE=16` 通常不会产生太多压缩效果，因为正常的 `InnoDB` 页大小为 16KB。对于具有许多长 `BLOB`、`VARCHAR` 或 `TEXT` 列的表，此设置仍然可能很有用，因为这些值通常可以很好地压缩，因此可能需要较少的溢出页，如 Section 17.9.1.5, “How Compression Works for InnoDB Tables” 中所述。对于通用表空间，不允许将 `KEY_BLOCK_SIZE` 值设置为 `InnoDB` 页大小。有关更多信息，请参见 Section 17.6.3.3, “General Tablespaces”。

+   表的所有索引（包括聚簇索引 语句的输出中）。

+   有关与性能相关的配置选项，请参见 Section 17.9.1.3, “Tuning Compression for InnoDB Tables”。

##### 压缩表的限制

+   压缩表不能存储在 `InnoDB` 系统表空间中。

+   通用表空间可以包含多个表，但压缩表和非压缩表不能共存于同一通用表空间中。

+   压缩适用于整个表及其所有关联的索引，而不是单独的行，尽管子句名称为 `ROW_FORMAT`。

+   `InnoDB` 不支持压缩临时表。当启用 `innodb_strict_mode`（默认情况下）时，如果指定了 `ROW_FORMAT=COMPRESSED` 或 `KEY_BLOCK_SIZE`，则 `CREATE TEMPORARY TABLE` 会返回错误。如果禁用了 `innodb_strict_mode`，则会发出警告，并且临时表将使用非压缩的行格式创建。对临时表的 `ALTER TABLE` 操作也适用相同的限制。
