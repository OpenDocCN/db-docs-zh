# 12.8.7 在 INFORMATION_SCHEMA 搜索中使用排序规则

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-collation-information-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-collation-information-schema.html)

`INFORMATION_SCHEMA`表中的字符串列具有`utf8mb3_general_ci`排序规则，这是不区分大小写的。但是，对于与文件系统中表示的对象对应的值，例如数据库和表，在`INFORMATION_SCHEMA`字符串列中的搜索可以是区分大小写或不区分大小写，这取决于底层文件系统的特性和`lower_case_table_names`系统变量设置。例如，如果文件系统区分大小写，则搜索可能是区分大小写的。本节描述了这种行为以及如何在必要时进行修改。

假设一个查询在`SCHEMATA.SCHEMA_NAME`列中搜索`test`数据库。在 Linux 上，文件系统区分大小写，因此`SCHEMATA.SCHEMA_NAME`与`'test'`的比较匹配，但与`'TEST'`的比较不匹配：

```sql
mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA
       WHERE SCHEMA_NAME = 'test';
+-------------+
| SCHEMA_NAME |
+-------------+
| test        |
+-------------+

mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA
       WHERE SCHEMA_NAME = 'TEST';
Empty set (0.00 sec)
```

这些结果发生在将`lower_case_table_names`系统变量设置为 0 时。将`lower_case_table_names`设置为 1 或 2 会导致第二个查询返回与第一个查询相同（非空）的结果。

注意

禁止使用与服务器初始化时使用的设置不同的`lower_case_table_names`设置启动服务器。

在 Windows 或 macOS 上，文件系统不区分大小写，因此比较匹配`'test'`和`'TEST'`：

```sql
mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA
       WHERE SCHEMA_NAME = 'test';
+-------------+
| SCHEMA_NAME |
+-------------+
| test        |
+-------------+

mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA
       WHERE SCHEMA_NAME = 'TEST';
+-------------+
| SCHEMA_NAME |
+-------------+
| TEST        |
+-------------+
```

在这种情况下，`lower_case_table_names`的值没有任何区别。

前面的行为发生是因为在搜索与文件系统中表示的对象对应的值时，`utf8mb3_general_ci`排序规则不用于`INFORMATION_SCHEMA`查询。

如果对`INFORMATION_SCHEMA`列的字符串操作的结果与预期不符，则可以使用显式的`COLLATE`子句来强制使用合适的排序规则（参见第 12.8.1 节，“在 SQL 语句中使用 COLLATE”）。例如，要执行不区分大小写的搜索，请在`INFORMATION_SCHEMA`列名后使用`COLLATE`：

```sql
mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA
       WHERE SCHEMA_NAME COLLATE utf8mb3_general_ci = 'test';
+-------------+
| SCHEMA_NAME |
+-------------+
| test        |
+-------------+

mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA
       WHERE SCHEMA_NAME COLLATE utf8mb3_general_ci = 'TEST';
+-------------+
| SCHEMA_NAME |
+-------------+
| test        |
+-------------+
```

你也可以使用`UPPER()`或`LOWER()`函数：

```sql
WHERE UPPER(SCHEMA_NAME) = 'TEST'
WHERE LOWER(SCHEMA_NAME) = 'test'
```

尽管即使在具有区分大小写文件系统的平台上也可以执行不区分大小写的比较，如上所示，但这并不一定总是正确的做法。在这样的平台上，可能存在仅在大小写不同的名称的多个对象。例如，同时存在名为`city`、`CITY`和`City`的表。考虑搜索是否应匹配所有这些名称或仅匹配一个，并相应地编写查询。以下比较中的第一个（使用`utf8mb3_bin`）是区分大小写的；其他则不是：

```sql
WHERE TABLE_NAME COLLATE utf8mb3_bin = 'City'
WHERE TABLE_NAME COLLATE utf8mb3_general_ci = 'city'
WHERE UPPER(TABLE_NAME) = 'CITY'
WHERE LOWER(TABLE_NAME) = 'city'
```

在`INFORMATION_SCHEMA`的字符串列中搜索引用`INFORMATION_SCHEMA`本身的值时，会使用`utf8mb3_general_ci`校对规则，因为`INFORMATION_SCHEMA`是一个在文件系统中没有实际表示的“虚拟”数据库。例如，与`SCHEMATA.SCHEMA_NAME`的比较会匹配`'information_schema'`或`'INFORMATION_SCHEMA'`，不受平台影响：

```sql
mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA
       WHERE SCHEMA_NAME = 'information_schema';
+--------------------+
| SCHEMA_NAME        |
+--------------------+
| information_schema |
+--------------------+

mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA
       WHERE SCHEMA_NAME = 'INFORMATION_SCHEMA';
+--------------------+
| SCHEMA_NAME        |
+--------------------+
| information_schema |
+--------------------+
```
