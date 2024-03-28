# 10.4.6 表大小限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/table-size-limit.html`](https://dev.mysql.com/doc/refman/8.0/en/table-size-limit.html)

MySQL 数据库的有效最大表大小通常由操作系统对文件大小的限制确定，而不是由 MySQL 内部限制确定。有关操作系统文件大小限制的最新信息，请参考特定于您操作系统的文档。

Windows 用户请注意，FAT 和 VFAT（FAT32）*不*被认为适用于与 MySQL 一起生产使用。请改用 NTFS。

如果遇到整个表错误，可能有几个原因：

+   磁盘可能已满。

+   您正在使用`InnoDB`表，并且在`InnoDB`表空间文件中已用完空间。表空间大小的最大值也是表的最大大小。有关表空间大小限制，请参阅 Section 17.22, “InnoDB Limits”。

    通常，对于大于 1TB 的表，建议将表分区到多个表空间文件中。

+   您已达到操作系统文件大小限制。例如，您正在使用`MyISAM`表，但操作系统仅支持最大为 2GB 的文件，并且您已达到数据文件或索引文件的限制。

+   您正在使用`MyISAM`表，表所需的空间超过了内部指针大小允许的范围。`MyISAM`默认允许数据和索引文件增长到 256TB，但此限制可更改为最大允许的大小 65,536TB（256⁷ − 1 字节）。

    如果您需要一个大于默认限制的`MyISAM`表，并且您的操作系统支持大文件，`CREATE TABLE`语句支持`AVG_ROW_LENGTH`和`MAX_ROWS`选项。请参阅 Section 15.1.20, “CREATE TABLE Statement”。服务器使用这些选项来确定允许多大的表。

    如果现有表的指针大小太小，您可以使用`ALTER TABLE`更改选项以增加表的最大允许大小。请参阅 Section 15.1.9, “ALTER TABLE Statement”。

    ```sql
    ALTER TABLE *tbl_name* MAX_ROWS=1000000000 AVG_ROW_LENGTH=*nnn*;
    ```

    您只需为具有`BLOB`或`TEXT`列的表指定`AVG_ROW_LENGTH`；在这种情况下，MySQL 无法仅基于行数优化所需的空间。

    要更改`MyISAM`表的默认大小限制，请设置`myisam_data_pointer_size`，该变量设置内部行指针使用的字节数。如果您没有指定`MAX_ROWS`选项，该值将用于设置新表的指针大小。`myisam_data_pointer_size`的值可以从 2 到 7。例如，对于使用动态存储格式的表，值为 4 允许表达到 4GB；值为 6 允许表达到 256TB。使用固定存储格式的表具有更大的最大数据长度。有关存储格式特性，请参见第 18.2.3 节，“MyISAM 表存储格式”。

    您可以使用以下语句检查最大数据和索引大小：

    ```sql
    SHOW TABLE STATUS FROM *db_name* LIKE '*tbl_name*';
    ```

    您还可以使用**myisamchk -dv /path/to/table-index-file**。参见第 15.7.7 节，“SHOW 语句”，或第 6.6.4 节，“myisamchk — MyISAM 表维护实用程序”。

    解决`MyISAM`表文件大小限制的其他方法如下：

    +   如果您的大表是只读的，您可以使用**myisampack**进行压缩。**myisampack**通常将表压缩至少 50%，因此您实际上可以拥有更大的表。**myisampack**还可以将多个表合并为单个表。参见第 6.6.6 节，“myisampack — 生成压缩的只读 MyISAM 表”。

    +   MySQL 包含一个`MERGE`库，使您能够将具有相同结构的一组`MyISAM`表作为单个`MERGE`表处理。参见第 18.7 节，“MERGE 存储引擎”。

+   你正在使用`MEMORY`（`HEAP`）存储引擎；在这种情况下，您需要增加`max_heap_table_size`系统变量的值。参见第 7.1.8 节，“服务器系统变量”。
