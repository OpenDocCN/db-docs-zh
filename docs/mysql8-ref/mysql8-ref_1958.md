# 28.4.12 INFORMATION_SCHEMA INNODB_FOREIGN 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-foreign-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-foreign-table.html)

[`INNODB_FOREIGN`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-foreign-table.html "28.4.12 INFORMATION_SCHEMA INNODB_FOREIGN 表") 表提供关于 `InnoDB` [外键](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_foreign_key "外键") 的元数据。

有关相关用法信息和示例，请参见第 17.15.3 节，“InnoDB INFORMATION_SCHEMA Schema Object Tables”。

[`INNODB_FOREIGN`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-foreign-table.html "28.4.12 INFORMATION_SCHEMA INNODB_FOREIGN 表") 表包含以下列：

+   `ID`

    外键索引的名称（不是数值），前面是模式（数据库）名称（例如，`test/products_fk`）。

+   `FOR_NAME`

    在这个外键关系中的[子表](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_child_table "子表") 的名称。

+   `REF_NAME`

    在这个外键关系中的[父表](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_parent_table "父表") 的名称。

+   `N_COLS`

    外键索引中的列数。

+   `TYPE`

    一个包含有关外键列信息的位标志集合，通过 OR 运算在一起。0 = `ON DELETE/UPDATE RESTRICT`，1 = `ON DELETE CASCADE`，2 = `ON DELETE SET NULL`，4 = `ON UPDATE CASCADE`，8 = `ON UPDATE SET NULL`，16 = `ON DELETE NO ACTION`，32 = `ON UPDATE NO ACTION`。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FOREIGN\G
*************************** 1\. row ***************************
      ID: test/fk1
FOR_NAME: test/child
REF_NAME: test/parent
  N_COLS: 1
    TYPE: 1
```

#### 注意

+   您必须具有[`PROCESS`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_process) 权限才能查询此表。

+   使用 `INFORMATION_SCHEMA` 的 [`COLUMNS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-columns-table.html "28.3.8 INFORMATION_SCHEMA COLUMNS 表") 表或 [`SHOW COLUMNS`](https://dev.mysql.com/doc/refman/8.0/en/show-columns.html "15.7.7.5 SHOW COLUMNS 语句") 语句查看有关此表的列的其他信息，包括数据类型和默认值。
