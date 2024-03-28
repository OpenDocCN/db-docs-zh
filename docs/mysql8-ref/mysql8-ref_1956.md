# 28.4.10 The INFORMATION_SCHEMA INNODB_DATAFILES Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-datafiles-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-datafiles-table.html)

`INNODB_DATAFILES`表提供了`InnoDB` file-per-table 和 general 表空间的数据文件路径信息。

有关使用信息和示例，请参见第 17.15.3 节，“InnoDB INFORMATION_SCHEMA Schema Object Tables”。

注意

`INFORMATION_SCHEMA` `FILES`表报告了`InnoDB`表空间类型的元数据，包括 file-per-table 表空间、general 表空间、系统表空间、全局临时表空间和撤销表空间。

`INNODB_DATAFILES`表具有以下列：

+   `SPACE`

    表空间 ID。

+   `PATH`

    表空间数据文件路径。如果在 MySQL 数据目录之外的位置创建了 file-per-table 表空间，则路径值是完全限定的目录路径。否则，路径是相对于数据目录的。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_DATAFILES WHERE SPACE = 57\G
*************************** 1\. row ***************************
SPACE: 57
 PATH: ./test/t1.ibd
```

#### 注意

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。
