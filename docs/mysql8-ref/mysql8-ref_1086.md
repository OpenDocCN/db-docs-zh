> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-create-table.html`](https://dev.mysql.com/doc/refman/8.0/en/show-create-table.html)

#### 15.7.7.10 SHOW CREATE TABLE Statement

```sql
SHOW CREATE TABLE *tbl_name*
```

显示创建指定表的 `CREATE TABLE` 语句。要使用此语句，您必须对该表具有某些权限。此语句也适用于视图。

```sql
mysql> SHOW CREATE TABLE t\G
*************************** 1\. row ***************************
       Table: t
Create Table: CREATE TABLE `t` (
  `id` int NOT NULL AUTO_INCREMENT,
  `s` char(60) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

从 MySQL 8.0.16 开始，MySQL 实现了 `CHECK` 约束，并且 `SHOW CREATE TABLE` 显示它们。所有 `CHECK` 约束都显示为表约束。也就是说，最初作为列定义的 `CHECK` 约束显示为一个独立的子句，而不是列定义的一部分。例如：

```sql
mysql> CREATE TABLE t1 (
         i1 INT CHECK (i1 <> 0),      -- column constraint
         i2 INT,
         CHECK (i2 > i1),             -- table constraint
         CHECK (i2 <> 0) NOT ENFORCED -- table constraint, not enforced
       );

mysql> SHOW CREATE TABLE t1\G
*************************** 1\. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `i1` int DEFAULT NULL,
  `i2` int DEFAULT NULL,
  CONSTRAINT `t1_chk_1` CHECK ((`i1` <> 0)),
  CONSTRAINT `t1_chk_2` CHECK ((`i2` > `i1`)),
  CONSTRAINT `t1_chk_3` CHECK ((`i2` <> 0)) /*!80016 NOT ENFORCED */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

`SHOW CREATE TABLE` 根据 `sql_quote_show_create` 选项的值引用表和列名。请参见 Section 7.1.8, “Server System Variables”。

当更改表的存储引擎时，不适用于新存储引擎的表选项将保留在表定义中，以便在必要时将具有先前定义选项的表还原到原始存储引擎。例如，当从 `InnoDB` 更改存储引擎为 `MyISAM` 时，特定于 `InnoDB` 的选项，如 `ROW_FORMAT=COMPACT`，将被保留，如下所示：

```sql
mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY) ROW_FORMAT=COMPACT ENGINE=InnoDB;
mysql> ALTER TABLE t1 ENGINE=MyISAM;
mysql> SHOW CREATE TABLE t1\G
*************************** 1\. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `c1` int NOT NULL,
  PRIMARY KEY (`c1`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci ROW_FORMAT=COMPACT
```

在禁用 严格模式 创建表时，如果指定的行格式不受支持，则使用存储引擎的默认行格式。表的实际行格式将在响应 `SHOW TABLE STATUS` 时的 `Row_format` 列中报告。`SHOW CREATE TABLE` 显示在 `CREATE TABLE` 语句中指定的行格式。

在 MySQL 8.0.30 及更高版本中，默认情况下，`SHOW CREATE TABLE` 包括表的生成的隐式主键的定义，如果表有这样的主键。您可以通过设置 `show_gipk_in_create_table_and_information_schema = OFF` 来使此信息在语句输出中被抑制。更多信息，请参见 Section 15.1.20.11, “Generated Invisible Primary Keys”。
