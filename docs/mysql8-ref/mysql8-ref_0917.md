> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-table-gipks.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table-gipks.html)

#### 15.1.20.11 生成的隐式主键

从 MySQL 8.0.30 开始，MySQL 支持为没有显式主键的任何`InnoDB`表创建生成的隐式主键。当`sql_generate_invisible_primary_key`服务器系统变量设置为`ON`时，MySQL 服务器会自动向任何这样的表添加一个生成的隐式主键（GIPK）。

默认情况下，`sql_generate_invisible_primary_key`的值为`OFF`，这意味着禁用 GIPK 的自动添加。为了说明这如何影响表的创建，我们首先创建两个相同的表，都没有主键，唯一的区别是第一个（表`auto_0`）在创建时`sql_generate_invisible_primary_key`设置为`OFF`，而第二个（`auto_1`）在将其设置为`ON`后创建，如下所示：

```sql
mysql> SELECT @@sql_generate_invisible_primary_key;
+--------------------------------------+
| @@sql_generate_invisible_primary_key |
+--------------------------------------+
|                                    0 |
+--------------------------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE auto_0 (c1 VARCHAR(50), c2 INT);
Query OK, 0 rows affected (0.02 sec)

mysql> SET sql_generate_invisible_primary_key=ON;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@sql_generate_invisible_primary_key;
+--------------------------------------+
| @@sql_generate_invisible_primary_key |
+--------------------------------------+
|                                    1 |
+--------------------------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE auto_1 (c1 VARCHAR(50), c2 INT);
Query OK, 0 rows affected (0.04 sec)
```

比较这些`SHOW CREATE TABLE`语句的输出，看看表实际上是如何创建的：

```sql
mysql> SHOW CREATE TABLE auto_0\G
*************************** 1\. row ***************************
       Table: auto_0
Create Table: CREATE TABLE `auto_0` (
  `c1` varchar(50) DEFAULT NULL,
  `c2` int DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.00 sec)

mysql> SHOW CREATE TABLE auto_1\G
*************************** 1\. row ***************************
       Table: auto_1
Create Table: CREATE TABLE `auto_1` (
  `my_row_id` bigint unsigned NOT NULL AUTO_INCREMENT /*!80023 INVISIBLE */,
  `c1` varchar(50) DEFAULT NULL,
  `c2` int DEFAULT NULL,
  PRIMARY KEY (`my_row_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.00 sec)
```

由于`auto_1`在创建时未指定主键，设置`sql_generate_invisible_primary_key = ON`会导致 MySQL 向该表添加不可见列`my_row_id`和在该列上的主键。由于在创建`auto_0`时`sql_generate_invisible_primary_key`为`OFF`，因此在该表上没有进行此类添加。

当服务器向表添加主键时，列和键名始终为`my_row_id`。因此，在以这种方式启用生成的隐式主键时，除非表创建语句还指定了显式主键，否则不能创建具有列名为`my_row_id`的表。（在这种情况下，您不需要在列或键中命名为`my_row_id`。）

`my_row_id`是一个不可见列，这意味着它不会显示在`SELECT *`或`TABLE`的输出中；必须通过名称显式选择该列。请参阅第 15.1.20.10 节，“不可见列”。

启用 GIPK 时，生成的主键除了在`VISIBLE`和`INVISIBLE`之间切换之外，不能进行其他更改。要使`auto_1`上的生成的隐式主键可见，请执行此`ALTER TABLE`语句：

```sql
mysql> ALTER TABLE auto_1 ALTER COLUMN my_row_id SET VISIBLE;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE auto_1\G
*************************** 1\. row ***************************
       Table: auto_1
Create Table: CREATE TABLE `auto_1` (
  `my_row_id` bigint unsigned NOT NULL AUTO_INCREMENT,
  `c1` varchar(50) DEFAULT NULL,
  `c2` int DEFAULT NULL,
  PRIMARY KEY (`my_row_id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.01 sec)
```

要再次使此生成的主键不可见，请执行`ALTER TABLE auto_1 ALTER COLUMN my_row_id SET INVISIBLE`。

生成的隐式主键默认情况下始终是不可见的。

启用 GIPK 时，如果满足以下任一条件，则无法删除生成的主键：

+   表没有主键。

+   主键被删除，但主键列未被删除。

`sql_generate_invisible_primary_key`的影响仅适用于使用`InnoDB`存储引擎的表。您可以使用`ALTER TABLE`语句更改表使用的存储引擎，该表具有生成的不可见主键；在这种情况下，主键和列保持不变，但表和键不再接受任何特殊处理。

默认情况下，GIPK 会显示在`SHOW CREATE TABLE`、`SHOW COLUMNS`和`SHOW INDEX`的输出中，并且在信息模式的`COLUMNS`和`STATISTICS`表中可见。您可以通过将`show_gipk_in_create_table_and_information_schema`系统变量设置为`OFF`来隐藏这些情况下生成的不可见主键。默认情况下，此变量为`ON`，如下所示：

```sql
mysql> SELECT @@show_gipk_in_create_table_and_information_schema;
+----------------------------------------------------+
| @@show_gipk_in_create_table_and_information_schema |
+----------------------------------------------------+
|                                                  1 |
+----------------------------------------------------+
1 row in set (0.00 sec)
```

如下查询`COLUMNS`表可见，`my_row_id`在`auto_1`的列中可见：

```sql
mysql> SELECT COLUMN_NAME, ORDINAL_POSITION, DATA_TYPE, COLUMN_KEY
 -> FROM INFORMATION_SCHEMA.COLUMNS
 -> WHERE TABLE_NAME = "auto_1";
+-------------+------------------+-----------+------------+
| COLUMN_NAME | ORDINAL_POSITION | DATA_TYPE | COLUMN_KEY |
+-------------+------------------+-----------+------------+
| my_row_id   |                1 | bigint    | PRI        |
| c1          |                2 | varchar   |            |
| c2          |                3 | int       |            |
+-------------+------------------+-----------+------------+
3 rows in set (0.01 sec)
```

在将`show_gipk_in_create_table_and_information_schema`设置为`OFF`后，`my_row_id`不再在`COLUMNS`表中可见，如下所示：

```sql
mysql> SET show_gipk_in_create_table_and_information_schema = OFF;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@show_gipk_in_create_table_and_information_schema;
+----------------------------------------------------+
| @@show_gipk_in_create_table_and_information_schema |
+----------------------------------------------------+
|                                                  0 |
+----------------------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT COLUMN_NAME, ORDINAL_POSITION, DATA_TYPE, COLUMN_KEY
 -> FROM INFORMATION_SCHEMA.COLUMNS
 -> WHERE TABLE_NAME = "auto_1";
+-------------+------------------+-----------+------------+
| COLUMN_NAME | ORDINAL_POSITION | DATA_TYPE | COLUMN_KEY |
+-------------+------------------+-----------+------------+
| c1          |                2 | varchar   |            |
| c2          |                3 | int       |            |
+-------------+------------------+-----------+------------+
2 rows in set (0.00 sec)
```

`sql_generate_invisible_primary_key`的设置不会被复制，并且被复制的应用程序线程会忽略它。这意味着在源上设置此变量对副本没有影响。在 MySQL 8.0.32 及更高版本中，您可以通过在`CHANGE REPLICATION SOURCE TO`语句中使用`REQUIRE_TABLE_PRIMARY_KEY_CHECK = GENERATE`来使副本在给定复制通道上为没有主键的表添加 GIPK。

GIPK 与基于行的`CREATE TABLE ... SELECT`复制一起工作；在这种情况下，写入二进制日志的信息包括 GIPK 定义，因此正确复制。不支持`sql_generate_invisible_primary_key = ON`的基于语句的`CREATE TABLE ... SELECT`复制。

在创建或导入使用 GIPK 的安装备份时，可以排除生成的不可见主键列和值。`--skip-generated-invisible-primary-key` 选项用于 **mysqldump**，导致在程序输出中排除 GIPK 信息。如果您正在导入包含生成的不可见主键和值的转储文件，还可以使用 `--skip-generated-invisible-primary-key` 与 **mysqlpump** 一起使用，以使这些信息被抑制（因此不会被导入）。
