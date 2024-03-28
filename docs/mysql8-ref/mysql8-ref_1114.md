> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-table-status.html`](https://dev.mysql.com/doc/refman/8.0/en/show-table-status.html)

#### 15.7.7.38 SHOW TABLE STATUS 语句

```sql
SHOW TABLE STATUS
    [{FROM | IN} *db_name*]
    [LIKE '*pattern*' | WHERE *expr*]
```

`SHOW TABLE STATUS` 类似于 `SHOW TABLES`，但提供有关每个非`TEMPORARY`表的大量信息。您还可以使用 **mysqlshow --status *`db_name`*** 命令获取此列表。如果存在 `LIKE` 子句，则指示要匹配的表名。`WHERE` 子句可以用于使用更一般的条件选择行，如 第 28.8 节 “SHOW 语句的扩展” 中所讨论的。

此语句还显示有关视图的信息。

`SHOW TABLE STATUS` 输出包括以下列：

+   `Name`

    表的名称。

+   `Engine`

    表的存储引擎。请参阅 第十七章 *InnoDB 存储引擎* 和 第十八章 *替代存储引擎*。

    对于分区表，`Engine` 显示所有分区使用的存储引擎的名称。

+   `Version`

    此列未使用。随着 MySQL 8.0 中 `.frm` 文件的移除，此列现在报告一个硬编码值 `10`，这是 MySQL 5.7 中使用的最后一个 `.frm` 文件版本。

+   `Row_format`

    行存储格式（`Fixed`、`Dynamic`、`Compressed`、`Redundant`、`Compact`）。对于 `MyISAM` 表，`Dynamic` 对应于 **myisamchk -dvv** 报告的 `Packed`。

+   `Rows`

    行数。一些存储引擎，如`MyISAM`，存储确切的计数。对于其他存储引擎，如`InnoDB`，此值是一个近似值，可能与实际值相差 40% 到 50%。在这种情况下，使用 `SELECT COUNT(*)` 来获取准确的计数。

    对于 `INFORMATION_SCHEMA` 表，`Rows` 值为 `NULL`。

    对于 `InnoDB` 表，行数仅是 SQL 优化中使用的粗略估计。（如果 `InnoDB` 表被分区，这也是正确的。）

+   `Avg_row_length`

    平均行长度。

+   `Data_length`

    对于 `MyISAM`，`Data_length` 是数据文件的长度，以字节为单位。

    对于 `InnoDB`，`Data_length` 是为聚簇索引分配的空间的近似量，以字节为单位。具体来说，它是聚簇索引大小（以页为单位）乘以 `InnoDB` 页大小。

    有关其他存储引擎的信息，请参考本节末尾的注释。

+   `Max_data_length`

    对于`MyISAM`，`Max_data_length`是数据文件的最大长度。这是可以存储在表中的数据字节数总数，考虑到使用的数据指针大小。

    对于`InnoDB`不适用。

    有关其他存储引擎的信息，请参考本节末尾的注释。

+   `Index_length`

    对于`MyISAM`，`Index_length`是索引文件的长度，以字节为单位。

    对于`InnoDB`，`Index_length`是非聚簇索引分配的大致空间量，以字节为单位。具体来说，它是非聚簇索引大小（以页为单位）的总和，乘以`InnoDB`页大小。

    有关其他存储引擎的信息，请参考本节末尾的注释。

+   `Data_free`

    已分配但未使用字节数。

    `InnoDB`表报告表所属表空间的可用空间。对于位于共享表空间中的表，这是共享表空间的可用空间。如果您使用多个表空间并且表有自己的表空间，则可用空间仅针对该表。可用空间指完全空闲的区段字节数减去安全边界。即使可用空间显示为 0，也可能可以插入行，只要不需要分配新的区段。

    对于 NDB Cluster，`Data_free`显示为磁盘上为磁盘数据表或片段分配但未使用的空间。（内存数据资源使用由`Data_length`列报告。）

    对于分区表，此值仅为估计值，可能不完全正确。在这种情况下获取此信息的更准确方法是查询`INFORMATION_SCHEMA` `PARTITIONS`表，如本例所示：

    ```sql
    SELECT SUM(DATA_FREE)
        FROM  INFORMATION_SCHEMA.PARTITIONS
        WHERE TABLE_SCHEMA = 'mydb'
        AND   TABLE_NAME   = 'mytable';
    ```

    有关更多信息，请参见第 28.3.21 节，“INFORMATION_SCHEMA PARTITIONS 表”。

+   `Auto_increment`

    下一个`AUTO_INCREMENT`值。

+   `Create_time`

    表创建时间。

+   `Update_time`

    数据文件上次更新时间。对于某些存储引擎，此值为`NULL`。例如，`InnoDB`在其系统表空间中存储多个表，数据文件时间戳不适用。即使每个`InnoDB`表在单独的`.ibd`文件中使用 file-per-table 模式，change buffering 也可以延迟对数据文件的写入，因此文件修改时间与最后一次插入、更新或删除的时间不同。对于`MyISAM`，使用数据文件时间戳；但是在 Windows 上，时间戳不会被更新，因此该值不准确。

    `Update_time`显示了对未分区的`InnoDB`表执行的最后一次`UPDATE`、`INSERT`或`DELETE`的时间戳值。对于 MVCC，时间戳值反映了`COMMIT`时间，被视为最后更新时间。当服务器重新启动或表从`InnoDB`数据字典缓存中删除时，时间戳不会被持久化。

+   `Check_time`

    上次检查表的时间。并非所有存储引擎都更新此时间，此时值始终为`NULL`。

    对于分区`InnoDB`表，`Check_time`始终为`NULL`。

+   `校对规则`

    表的默认校对规则。输出不明确列出表的默认字符集，但校对规则名称以字符集名称开头。

+   `校验和`

    实时校验和值（如果有）。

+   `Create_options`

    与`CREATE TABLE`一起使用的额外选项。

    对于分区表，`Create_options`显示`partitioned`。

    在 MySQL 8.0.16 之前，对于在文件表空间中创建的表，`Create_options`显示指定的`ENCRYPTION`子句。从 MySQL 8.0.16 开始，如果表已加密或指定的加密与模式加密不同，则显示文件表空间的加密子句。对于在一般表空间中创建的表，不显示加密子句。要识别加密的文件表空间和一般表空间，请查询`INNODB_TABLESPACES`的`ENCRYPTION`列。

    在禁用严格模式创建表时，如果指定的行格式不受支持，则使用存储引擎的默认行格式。表的实际行格式在`Row_format`列中报告。`Create_options`显示了在`CREATE TABLE`语句中指定的行格式。

    当更改表的存储引擎时，不适用于新存储引擎的表选项将保留在表定义中，以便在必要时将表及其先前定义的选项还原为原始存储引擎。`Create_options`可能显示保留的选项。

+   `注释`

    创建表时使用的注释（或 MySQL 无法访问表信息的原因）。

##### 备注

+   对于`InnoDB`表，`SHOW TABLE STATUS`除了表所保留的物理大小外，不提供准确的统计信息。行数仅是 SQL 优化中使用的粗略估计。

+   对于`NDB`表，此语句的输出显示了`Avg_row_length`和`Data_length`列的适当值，但不考虑`BLOB`列。

+   对于`NDB`表，`Data_length`仅包括存储在主内存中的数据；`Max_data_length`和`Data_free`列适用于磁盘数据。

+   对于 NDB 集群磁盘数据表，`Max_data_length`显示为磁盘数据表或片段的磁盘部分分配的空间。（内存数据资源使用情况由`Data_length`列报告。）

+   对于`MEMORY`表，`Data_length`、`Max_data_length`和`Index_length`的值近似表示实际分配的内存量。分配算法会大量保留内存以减少分配操作的次数。

+   对于视图，`SHOW TABLE STATUS`显示的大多数列都为 0 或`NULL`，除了`Name`表示视图名称，`Create_time`表示创建时间，`Comment`显示为`VIEW`。

表信息也可以从`INFORMATION_SCHEMA` `TABLES`表中获取。请参见第 28.3.38 节，“INFORMATION_SCHEMA TABLES 表”。
