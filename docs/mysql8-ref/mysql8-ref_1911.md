# 28.3.16 INFORMATION_SCHEMA KEY_COLUMN_USAGE 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-key-column-usage-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-key-column-usage-table.html)

`KEY_COLUMN_USAGE` 表描述了哪些关键列具有约束。该表不提供有关功能键部分的信息，因为它们是表达式，而该表仅提供有关列的信息。

`KEY_COLUMN_USAGE` 表具有以下列：

+   `CONSTRAINT_CATALOG`

    约束所属的目录的名称。此值始终为`def`。

+   `CONSTRAINT_SCHEMA`

    约束所属的模式（数据库）的名称。

+   `CONSTRAINT_NAME`

    约束的名称。

+   `TABLE_CATALOG`

    表所属的目录的名称。此值始终为`def`。

+   `TABLE_SCHEMA`

    表所属的模式（数据库）的名称。

+   `TABLE_NAME`

    具有约束的表的名称。

+   `COLUMN_NAME`

    具有约束的列的名称。

    如果约束是外键，则这是外键的列，而不是外键引用的列。

+   `ORDINAL_POSITION`

    列在约束中的位置，而不是表中的列位置。列位置从 1 开始编号。

+   `POSITION_IN_UNIQUE_CONSTRAINT`

    对于唯一约束和主键约束为`NULL`。对于外键约束，此列是被引用的表中键的序数位置。

+   `REFERENCED_TABLE_SCHEMA`

    约束引用的模式的名称。

+   `REFERENCED_TABLE_NAME`

    约束引用的表的名称。

+   `REFERENCED_COLUMN_NAME`

    约束引用的列的名称。

假设有两个名为`t1`和`t3`的表，其定义如下：

```sql
CREATE TABLE t1
(
    s1 INT,
    s2 INT,
    s3 INT,
    PRIMARY KEY(s3)
) ENGINE=InnoDB;

CREATE TABLE t3
(
    s1 INT,
    s2 INT,
    s3 INT,
    KEY(s1),
    CONSTRAINT CO FOREIGN KEY (s2) REFERENCES t1(s3)
) ENGINE=InnoDB;
```

对于这两个表，`KEY_COLUMN_USAGE` 表有两行：

+   具有`CONSTRAINT_NAME` = `'PRIMARY'`、`TABLE_NAME` = `'t1'`、`COLUMN_NAME` = `'s3'`、`ORDINAL_POSITION` = `1`、`POSITION_IN_UNIQUE_CONSTRAINT` = `NULL`的一行。

    对于`NDB`：此值始终为`NULL`。

+   具有`CONSTRAINT_NAME` = `'CO'`、`TABLE_NAME` = `'t3'`、`COLUMN_NAME` = `'s2'`、`ORDINAL_POSITION` = `1`、`POSITION_IN_UNIQUE_CONSTRAINT` = `1`的一行。
