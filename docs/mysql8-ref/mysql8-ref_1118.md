> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-warnings.html`](https://dev.mysql.com/doc/refman/8.0/en/show-warnings.html)

#### 15.7.7.42 SHOW WARNINGS Statement

```sql
SHOW WARNINGS [LIMIT [*offset*,] *row_count*]
SHOW COUNT(*) WARNINGS
```

`SHOW WARNINGS` 是一个诊断性语句，显示关于当前会话中执行语句产生的条件（错误、警告和注释）的信息。警告会为诸如 `INSERT`、`UPDATE` 和 `LOAD DATA` 等 DML 语句以及 `CREATE TABLE` 和 `ALTER TABLE` 等 DDL 语句生成。

`LIMIT` 子句与 `SELECT` 语句具有相同的语法。参见 Section 15.2.13, “SELECT Statement”。

`SHOW WARNINGS` 也用于在 `EXPLAIN` 之后，显示由 `EXPLAIN` 生成的扩展信息。参见 Section 10.8.3, “Extended EXPLAIN Output Format”。

`SHOW WARNINGS` 显示关于当前会话中最近一次非诊断性语句执行结果的条件信息。如果最近的语句在解析过程中出现错误，`SHOW WARNINGS` 将显示结果的条件，无论语句类型（诊断性或非诊断性）如何。

`SHOW COUNT(*) WARNINGS` 诊断性语句显示错误、警告和注释的总数。您还可以从 `warning_count` 系统变量中检索此数字：

```sql
SHOW COUNT(*) WARNINGS;
SELECT @@warning_count;
```

这些语句的区别在于第一个是一个不清除消息列表的诊断性语句。第二个，因为是一个 `SELECT` 语句，被视为非诊断性语句并清除消息列表。

相关的诊断语句`SHOW ERRORS`仅显示错误条件（排除警告和注释），而`SHOW COUNT(*) ERRORS`语句显示错误的总数。请参阅 Section 15.7.7.17, “SHOW ERRORS Statement”。`GET DIAGNOSTICS`可用于检查各个条件的信息。请参阅 Section 15.6.7.3, “GET DIAGNOSTICS Statement”。

这里有一个简单的示例，显示了`INSERT`的数据转换警告。该示例假定严格的 SQL 模式已禁用。启用严格模式后，警告将变为错误，并终止`INSERT`。

```sql
mysql> CREATE TABLE t1 (a TINYINT NOT NULL, b CHAR(4));
Query OK, 0 rows affected (0.05 sec)

mysql> INSERT INTO t1 VALUES(10,'mysql'), (NULL,'test'), (300,'xyz');
Query OK, 3 rows affected, 3 warnings (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 3

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Warning
   Code: 1265
Message: Data truncated for column 'b' at row 1
*************************** 2\. row ***************************
  Level: Warning
   Code: 1048
Message: Column 'a' cannot be null
*************************** 3\. row ***************************
  Level: Warning
   Code: 1264
Message: Out of range value for column 'a' at row 3 3 rows in set (0.00 sec)
```

`max_error_count`系统变量控制服务器存储信息的最大错误、警告和注释消息数量，因此也控制`SHOW WARNINGS`显示的消息数量。要更改服务器可以存储的消息数量，请更改`max_error_count`的值。

`max_error_count`仅控制存储的消息数量，而不是计数的数量。即使生成的消息数量超过`max_error_count`，`warning_count`的值也不受`max_error_count`的限制。以下示例演示了这一点。`ALTER TABLE`语句生成三条警告消息（示例中已禁用严格的 SQL 模式，以防止在单个转换问题后发生错误）。只有一条消息被存储和显示，因为`max_error_count`已设置为 1，但所有三条都被计数（如`warning_count`的值所示）：

```sql
mysql> SHOW VARIABLES LIKE 'max_error_count';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_error_count | 1024  |
+-----------------+-------+
1 row in set (0.00 sec)

mysql> SET max_error_count=1, sql_mode = '';
Query OK, 0 rows affected (0.00 sec)

mysql> ALTER TABLE t1 MODIFY b CHAR;
Query OK, 3 rows affected, 3 warnings (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 3

mysql> SHOW WARNINGS;
+---------+------+----------------------------------------+
| Level   | Code | Message                                |
+---------+------+----------------------------------------+
| Warning | 1263 | Data truncated for column 'b' at row 1 |
+---------+------+----------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT @@warning_count;
+-----------------+
| @@warning_count |
+-----------------+
|               3 |
+-----------------+
1 row in set (0.01 sec)
```

要禁用消息存储，请将`max_error_count`设置为 0。在这种情况下，`warning_count`仍然指示发生了多少警告，但消息不会被存储，也无法显示。

`sql_notes`系统变量控制注释消息是否会增加`warning_count`以及服务器是否会存储它们。默认情况下，`sql_notes`为 1，但如果设置为 0，则注释不会增加`warning_count`，服务器也不会存储它们：

```sql
mysql> SET sql_notes = 1;
mysql> DROP TABLE IF EXISTS test.no_such_table;
Query OK, 0 rows affected, 1 warning (0.00 sec)
mysql> SHOW WARNINGS;
+-------+------+------------------------------------+
| Level | Code | Message                            |
+-------+------+------------------------------------+
| Note  | 1051 | Unknown table 'test.no_such_table' |
+-------+------+------------------------------------+
1 row in set (0.00 sec)

mysql> SET sql_notes = 0;
mysql> DROP TABLE IF EXISTS test.no_such_table;
Query OK, 0 rows affected (0.00 sec)
mysql> SHOW WARNINGS;
Empty set (0.00 sec)
```

MySQL 服务器向每个客户端发送一个计数，指示由该客户端执行的最近语句导致的错误、警告和注释的总数。从 C API，可以通过调用`mysql_warning_count()`来获取此值。参见 mysql_warning_count()。

在**mysql**客户端中，可以使用`warnings`和`nowarning`命令或它们的快捷方式`\W`和`\w`（参见第 6.5.1.2 节，“mysql 客户端命令”

Warning (Code 1365): Division by 0
mysql> \w
Show warnings disabled.
```
