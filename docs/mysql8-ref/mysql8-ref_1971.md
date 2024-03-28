# 28.4.25 The INFORMATION_SCHEMA INNODB_TABLESPACES_BRIEF Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-tablespaces-brief-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-tablespaces-brief-table.html)

`INNODB_TABLESPACES_BRIEF`表为每个表、通用、撤销和系统表空间提供了文件表空间的空间 ID、名称、路径、标志和空间类型元数据。

`INNODB_TABLESPACES`提供相同的元数据，但加载速度较慢，因为表提供的其他元数据，如`FS_BLOCK_SIZE`、`FILE_SIZE`和`ALLOCATED_SIZE`，必须动态加载。

空间和路径元数据也由`INNODB_DATAFILES`表提供。

`INNODB_TABLESPACES_BRIEF`表具有以下列：

+   `SPACE`

    表空间 ID。

+   `NAME`

    表空间名称。对于每个表的文件表空间，名称采用*`schema/table_name`*的形式。

+   `PATH`

    表空间数据文件路径。如果在 MySQL 数据目录之外创建了 file-per-table 表空间，则路径值是完全限定的目录路径。否则，路径是相对于数据目录的。

+   `FLAG`

    代表表空间格式和存储特性的位级信息的数值。

+   `SPACE_TYPE`

    表空间类型。可能的值包括`General`表示`InnoDB`通用表空间，`Single`表示`InnoDB`每个表的文件表空间，`System`表示`InnoDB`系统表空间。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLESPACES_BRIEF WHERE SPACE = 7;
+-------+---------+---------------+-------+------------+
| SPACE | NAME    | PATH          | FLAG  | SPACE_TYPE |
+-------+---------+---------------+-------+------------+
| 7     | test/t1 | ./test/t1.ibd | 16417 | Single     |
+-------+---------+---------------+-------+------------+
```

#### 笔记

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表列的其他信息，包括数据类型和默认值。
