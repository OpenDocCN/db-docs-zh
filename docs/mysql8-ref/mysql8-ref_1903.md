# 28.3.8 The INFORMATION_SCHEMA COLUMNS Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-columns-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-columns-table.html)

`COLUMNS` 表提供有关表中列的信息。相关的 `ST_GEOMETRY_COLUMNS` 表提供有关存储空间数据的表列的信息。请参阅 Section 28.3.35, “The INFORMATION_SCHEMA ST_GEOMETRY_COLUMNS Table”。

`COLUMNS` 表具有以下列：

+   `TABLE_CATALOG`

    包含列的表所属的目录名称。此值始终为 `def`。

+   `TABLE_SCHEMA`

    包含列的表所属的模式（数据库）的名称。

+   `TABLE_NAME`

    包含列的表的名称。

+   `COLUMN_NAME`

    列的名称。

+   `ORDINAL_POSITION`

    列在表中的位置。`ORDINAL_POSITION` 是必要的，因为您可能想说 `ORDER BY ORDINAL_POSITION`。与 `SHOW COLUMNS` 不同，从 `COLUMNS` 表中的 `SELECT` 没有自动排序。

+   `COLUMN_DEFAULT`

    列的默认值。如果列具有显式默认值为 `NULL`，或者列定义不包含 `DEFAULT` 子句，则为 `NULL`。

+   `IS_NULLABLE`

    列的可空性。如果可以在列中存储 `NULL` 值，则值为 `YES`，否则为 `NO`。

+   `DATA_TYPE`

    列数据类型。

    `DATA_TYPE` 值仅为类型名称，没有其他信息。`COLUMN_TYPE` 值包含类型名称，可能还包含其他信息，如精度或长度。

+   `CHARACTER_MAXIMUM_LENGTH`

    对于字符串列，以字符为单位的最大长度。

+   `CHARACTER_OCTET_LENGTH`

    对于字符串列，以字节为单位的最大长度。

+   `NUMERIC_PRECISION`

    对于数字列，数字精度。

+   `NUMERIC_SCALE`

    对于数字列，数字刻度。

+   `DATETIME_PRECISION`

    对于时间列，小数秒精度。

+   `CHARACTER_SET_NAME`

    对于字符串列，字符集名称。

+   `COLLATION_NAME`

    对于字符串列，排序规则名称。

+   `COLUMN_TYPE`

    列数据类型。

    `DATA_TYPE` 值仅为类型名称，没有其他信息。`COLUMN_TYPE` 值包含类型名称，可能还包含其他信息，如精度或长度。

+   `COLUMN_KEY`

    列是否已索引：

    +   如果`COLUMN_KEY`为空，则该列未被索引或仅作为多列非唯一索引中的次要列被索引。

    +   如果`COLUMN_KEY`为`PRI`，则该列是`PRIMARY KEY`或是多列`PRIMARY KEY`中的一列。

    +   如果`COLUMN_KEY`为`UNI`，则该列是`UNIQUE`索引的第一列。（`UNIQUE`索引允许多个`NULL`值，但您可以通过检查`Null`列来确定列是否允许`NULL`。）

    +   如果`COLUMN_KEY`为`MUL`，则该列是非唯一索引的第一列，在该索引中允许在列内出现给定值的多个实例。

    如果表的给定列有多个`COLUMN_KEY`值适用，则`COLUMN_KEY`显示具有最高优先级的值，按照`PRI`，`UNI`，`MUL`的顺序。

    如果`UNIQUE`索引不能包含`NULL`值且表中没有`PRIMARY KEY`，则`UNIQUE`索引可能显示为`PRI`。如果几列形成复合`UNIQUE`索引，则`UNIQUE`索引可能显示为`MUL`；尽管列的组合是唯一的，但每列仍然可以包含给定值的多个实例。

+   `EXTRA`

    关于给定列的任何其他可用信息。在这些情况下，该值不为空：

    +   对于具有`AUTO_INCREMENT`属性的列，显示`auto_increment`。

    +   对于具有`ON UPDATE CURRENT_TIMESTAMP`属性的`TIMESTAMP`或`DATETIME`列，显示`on update CURRENT_TIMESTAMP`。

    +   对于生成列，显示`STORED GENERATED`或`VIRTUAL GENERATED`。

    +   `DEFAULT_GENERATED` 用于具有表达式默认值的列。

+   `PRIVILEGES`

    您对该列拥有的权限。

+   `COLUMN_COMMENT`

    包括在列定义中的任何注释。

+   `GENERATION_EXPRESSION`

    对于生成的列，显示用于计算列值的表达式。对于非生成列为空。有关生成列的信息，请参见第 15.1.20.8 节，“CREATE TABLE 和生成列”。

+   `SRS_ID`

    此值适用于空间列。它包含列`SRID`值，指示存储在列中的值的空间参考系统。参见第 13.4.1 节，“空间数据类型”和第 13.4.5 节，“空间参考系统支持”。对于非空间列和没有`SRID`属性的空间列，该值为`NULL`。

#### 注释

+   在`SHOW COLUMNS`中，`Type`显示包括来自几个不同的`COLUMNS`列的值。

+   `CHARACTER_OCTET_LENGTH`应与`CHARACTER_MAXIMUM_LENGTH`相同，除了多字节字符集。

+   `CHARACTER_SET_NAME`可以从`COLLATION_NAME`派生。例如，如果你执行`SHOW FULL COLUMNS FROM t`，并且在`COLLATION_NAME`列中看到一个值为`utf8mb4_swedish_ci`，那么字符集就是出现在第一个下划线之前的内容：`utf8mb4`。

列信息也可以通过`SHOW COLUMNS`语句获取。参见 Section 15.7.7.5, “SHOW COLUMNS Statement”。以下语句几乎是等效的：

```sql
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE, COLUMN_DEFAULT
  FROM INFORMATION_SCHEMA.COLUMNS
  WHERE table_name = '*tbl_name*'
  [AND table_schema = '*db_name*']
  [AND column_name LIKE '*wild*']

SHOW COLUMNS
  FROM *tbl_name*
  [FROM *db_name*]
  [LIKE '*wild*']
```

在 MySQL 8.0.30 及更高版本中，默认情况下，此表中可见生成的不可见主键列的信息。您可以通过设置`show_gipk_in_create_table_and_information_schema = OFF`来隐藏这些信息。更多信息，请参见 Section 15.1.20.11, “Generated Invisible Primary Keys”。
