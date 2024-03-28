# 28.4.9 INFORMATION_SCHEMA INNODB_COLUMNS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-columns-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-columns-table.html)

`INNODB_COLUMNS` 表提供关于 `InnoDB` 表列的元数据。

有关相关用法信息和示例，请参见 第 17.15.3 节，“InnoDB INFORMATION_SCHEMA Schema Object Tables”。

`INNODB_COLUMNS` 表具有以下列：

+   `TABLE_ID`

    表示与列关联的表的标识符；与 `INNODB_TABLES.TABLE_ID` 相同的值。

+   `NAME`

    列的名称。这些名称可以是大写或小写，取决于 `lower_case_table_names` 设置。列没有特殊的系统保留名称。

+   `POS`

    列在表中的序数位置，从 0 开始顺序递增。当删除列时，剩余列将重新排序，使序列没有间隙。虚拟生成列的 `POS` 值编码列序号和列的序数位置。有关更多信息，请参见 第 28.4.29 节，“INFORMATION_SCHEMA INNODB_VIRTUAL 表” 中的 `POS` 列描述。  

+   `MTYPE`

    代表“主类型”。列类型的数字标识符。1 = `VARCHAR`，2 = `CHAR`，3 = `FIXBINARY`，4 = `BINARY`，5 = `BLOB`，6 = `INT`，7 = `SYS_CHILD`，8 = `SYS`，9 = `FLOAT`，10 = `DOUBLE`，11 = `DECIMAL`，12 = `VARMYSQL`，13 = `MYSQL`，14 = `GEOMETRY`。

+   `PRTYPE`

    `InnoDB` 的“精确类型”，一个二进制值，其中位表示 MySQL 数据类型、字符集代码和可为 null。

+   `LEN`

    列长度，例如 `INT` 为 4，`BIGINT` 为 8。对于多字节字符集中的字符列，此长度值是以字节表示定义所需的最大长度，例如 `VARCHAR(*`N`*)`；也就是说，它可能是 `2**`N`*`，`3**`N`*`，依此类推，取决于字符编码。

+   `HAS_DEFAULT`

    一个布尔值，指示使用 `ALTER TABLE ... ADD COLUMN` 和 `ALGORITHM=INSTANT` 立即添加的列是否具有默认值。所有立即添加的列都有默认值，这使得此列成为指示列是否立即添加的指示器。

+   `DEFAULT_VALUE`

    使用`ALTER TABLE ... ADD COLUMN`立即添加的列的初始默认值，使用`ALGORITHM=INSTANT`。如果默认值为`NULL`或未指定，默认情况下此列报告`NULL`。显式指定的非`NULL`默认值以内部二进制格式显示。对列默认值的后续修改不会更改此列报告的值。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_COLUMNS where TABLE_ID = 71\G
*************************** 1\. row ***************************
     TABLE_ID: 71
         NAME: col1
          POS: 0
        MTYPE: 6
       PRTYPE: 1027
          LEN: 4
  HAS_DEFAULT: 0
DEFAULT_VALUE: NULL
*************************** 2\. row ***************************
     TABLE_ID: 71
         NAME: col2
          POS: 1
        MTYPE: 2
       PRTYPE: 524542
          LEN: 10
  HAS_DEFAULT: 0
DEFAULT_VALUE: NULL
*************************** 3\. row ***************************
     TABLE_ID: 71
         NAME: col3
          POS: 2
        MTYPE: 1
       PRTYPE: 524303
          LEN: 10
  HAS_DEFAULT: 0
DEFAULT_VALUE: NULL
```

#### 注意事项

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。
