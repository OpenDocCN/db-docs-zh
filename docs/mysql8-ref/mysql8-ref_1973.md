# 28.4.27 INFORMATION_SCHEMA INNODB_TEMP_TABLE_INFO 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-temp-table-info-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-temp-table-info-table.html)

`INNODB_TEMP_TABLE_INFO`表提供有关在`InnoDB`实例中活动的用户创建的`InnoDB`临时表的信息。它不提供有关优化器使用的内部`InnoDB`临时表的信息。`INNODB_TEMP_TABLE_INFO`表在首次查询时创建，仅存在于内存中，不会持久保存到磁盘。

有关用法信息和示例，请参见第 17.15.7 节，“InnoDB INFORMATION_SCHEMA 临时表信息表”。

`INNODB_TEMP_TABLE_INFO`表具有以下列：

+   `TABLE_ID`

    临时表的表 ID。

+   `NAME`

    临时表的名称。

+   `N_COLS`

    临时表中的列数。该数字包括`InnoDB`创建的三个隐藏列（`DB_ROW_ID`、`DB_TRX_ID`和`DB_ROLL_PTR`）。

+   `SPACE`

    临时表所在的临时表空间的 ID。

#### 示例

```sql
mysql> CREATE TEMPORARY TABLE t1 (c1 INT PRIMARY KEY) ENGINE=INNODB;

mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TEMP_TABLE_INFO\G
*************************** 1\. row ***************************
TABLE_ID: 97
    NAME: #sql8c88_43_0
  N_COLS: 4
   SPACE: 76
```

#### 注意

+   此表主要用于专家级别的监视。

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表列的其他信息，包括数据类型和默认值。
