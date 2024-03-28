# 28.3.32 `INFORMATION_SCHEMA SCHEMATA_EXTENSIONS` 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-schemata-extensions-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-schemata-extensions-table.html)

[`SCHEMATA_EXTENSIONS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-schemata-extensions-table.html) 表（自 MySQL 8.0.22 起可用）通过提供有关模式选项的信息来扩充 `SCHEMATA` 表。

[`SCHEMATA_EXTENSIONS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-schemata-extensions-table.html) 表包含以下列：

+   `CATALOG_NAME`

    模式所属的目录名称。该值始终为`def`。

+   `SCHEMA_NAME`

    模式的名称。

+   `OPTIONS`

    模式的选项。如果模式是只读的，则值包含`READ ONLY=1`。如果模式不是只读的，则不会出现`READ ONLY`选项。

#### 示例

```sql
mysql> ALTER SCHEMA mydb READ ONLY = 1;
mysql> SELECT * FROM INFORMATION_SCHEMA.SCHEMATA_EXTENSIONS
       WHERE SCHEMA_NAME = 'mydb';
+--------------+-------------+-------------+
| CATALOG_NAME | SCHEMA_NAME | OPTIONS     |
+--------------+-------------+-------------+
| def          | mydb        | READ ONLY=1 |
+--------------+-------------+-------------+

mysql> ALTER SCHEMA mydb READ ONLY = 0;
mysql> SELECT * FROM INFORMATION_SCHEMA.SCHEMATA_EXTENSIONS
       WHERE SCHEMA_NAME = 'mydb';
+--------------+-------------+---------+
| CATALOG_NAME | SCHEMA_NAME | OPTIONS |
+--------------+-------------+---------+
| def          | mydb        |         |
+--------------+-------------+---------+
```

#### 注意事项

+   [`SCHEMATA_EXTENSIONS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-schemata-extensions-table.html) 是一个非标准的`INFORMATION_SCHEMA`表。
