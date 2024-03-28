# 1.6.1 MySQL 对标准 SQL 的扩展

> 原文：[`dev.mysql.com/doc/refman/8.0/en/extensions-to-ansi.html`](https://dev.mysql.com/doc/refman/8.0/en/extensions-to-ansi.html)

MySQL 服务器支持一些其他 SQL DBMS 中不太可能找到的扩展。请注意，如果您使用它们，您的代码很可能无法在其他 SQL 服务器上移植。在某些情况下，您可以编写包含 MySQL 扩展的代码，但仍然是可移植的，方法是使用以下形式的注释：

```sql
/*! *MySQL-specific code* */
```

在这种情况下，MySQL 服务器会解析并执行注释中的代码，就像执行任何其他 SQL 语句一样，但其他 SQL 服务器应该忽略这些扩展。例如，MySQL 服务器会识别以下语句中的`STRAIGHT_JOIN`关键字，但其他服务器不应该：

```sql
SELECT /*! STRAIGHT_JOIN */ col1 FROM table1,table2 WHERE ...
```

如果在`!`字符后添加版本号，则仅当 MySQL 版本大于或等于指定版本号时才执行注释中的语法。以下注释中的`KEY_BLOCK_SIZE`子句仅在 MySQL 5.1.10 或更高版本的服务器上执行：

```sql
CREATE TABLE t1(a INT, KEY (a)) /*!50110 KEY_BLOCK_SIZE=1024 */;
```

以下描述列出了按类别组织的 MySQL 扩展。

+   数据在磁盘上的组织

    MySQL 服务器将每个数据库映射到 MySQL 数据目录下的一个目录，并将数据库中的表映射到数据库目录中的文件名。因此，在具有区分大小写文件名的操作系统上（例如大多数 Unix 系统），MySQL 服务器中的数据库和表名称是区分大小写的。请参阅 Section 11.2.3, “Identifier Case Sensitivity”。

+   通用语言语法

    +   默认情况下，字符串可以用`"`或`'`括起来。如果启用了`ANSI_QUOTES` SQL 模式，则字符串只能用`'`括起来，服务器会将用`"`括起来的字符串解释为标识符。

    +   `\`是字符串中的转义字符。

    +   在 SQL 语句中，您可以使用*`db_name.tbl_name`*语法访问不同数据库中的表。一些 SQL 服务器提供相同的功能，但称其为`User space`。MySQL 服务器不支持像这样使用表空间的语句：`CREATE TABLE ralph.my_table ... IN my_tablespace`。

+   SQL 语句语法

    +   `ANALYZE TABLE`、`CHECK TABLE`、`OPTIMIZE TABLE`和`REPAIR TABLE`语句。

    +   `CREATE DATABASE`、`DROP DATABASE`和`ALTER DATABASE`语句。参见第 15.1.12 节，“CREATE DATABASE Statement”，第 15.1.24 节，“DROP DATABASE Statement”和第 15.1.2 节，“ALTER DATABASE Statement”。

    +   `DO`语句。

    +   `EXPLAIN SELECT`以获取查询优化器如何处理表格的描述。

    +   `FLUSH`和`RESET`语句。

    +   `SET`语句。参见第 15.7.6.1 节，“SET Syntax for Variable Assignment”。

    +   `SHOW`语句。参见第 15.7.7 节，“SHOW Statements”。许多特定于 MySQL 的`SHOW`语句产生的信息可以通过使用`SELECT`查询`INFORMATION_SCHEMA`来更标准地获取。参见第二十八章，“INFORMATION_SCHEMA Tables”。

    +   使用`LOAD DATA`语句。在许多情况下，此语法与 Oracle 的`LOAD DATA`兼容。参见第 15.2.9 节，“LOAD DATA Statement”。

    +   使用`RENAME TABLE`语句。参见第 15.1.36 节，“RENAME TABLE Statement”。

    +   使用`REPLACE`代替`DELETE`加`INSERT`。参见第 15.2.12 节，“REPLACE Statement”。

    +   在`ALTER TABLE`语句中使用`CHANGE *`col_name`*`，`DROP *`col_name`*`，或`DROP INDEX`，`IGNORE`或`RENAME`。在`ALTER TABLE`语句中使用多个`ADD`，`ALTER`，`DROP`或`CHANGE`子句。参见第 15.1.9 节，“ALTER TABLE Statement”。

    +   在 `CREATE TABLE` 语句中使用索引名称、列前缀上的索引，以及使用 `INDEX` 或 `KEY`。参见 第 15.1.20 节，“CREATE TABLE Statement”。

    +   在 `CREATE TABLE` 中使用 `TEMPORARY` 或 `IF NOT EXISTS`。

    +   在 `DROP TABLE` 和 `DROP DATABASE` 中使用 `IF EXISTS`。

    +   使用单个 `DROP TABLE` 语句删除多个表的能力。

    +   `UPDATE` 和 `DELETE` 语句的 `ORDER BY` 和 `LIMIT` 子句。

    +   `INSERT INTO *`tbl_name`* SET *`col_name`* = ...` 语法。

    +   `INSERT` 和 `REPLACE` 语句的 `DELAYED` 子句。

    +   `INSERT`、`REPLACE`、`DELETE` 和 `UPDATE` 语句的 `LOW_PRIORITY` 子句。

    +   在 `SELECT` 语句中使用 `INTO OUTFILE` 或 `INTO DUMPFILE`。参见 第 15.2.13 节，“SELECT Statement”。

    +   在 `SELECT` 语句中的选项，如 `STRAIGHT_JOIN` 或 `SQL_SMALL_RESULT`。

    +   在 `GROUP BY` 子句中不需要命名所有选定的列。这对于一些非常特定但相当常见的查询可以提供更好的性能。参见 第 14.19 节，“聚合函数”。

    +   您可以在 `GROUP BY` 中指定 `ASC` 和 `DESC`，而不仅仅是在 `ORDER BY` 中。

    +   使用 `:=` 赋值运算符在语句中设置变量的能力��参见 第 11.4 节，“用户定义变量”。

+   数据类型

    +   `MEDIUMINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")、`SET` 和 `ENUM` 数据类型，以及各种 `BLOB` 和 `TEXT` 数据类型。

    +   `AUTO_INCREMENT`、`BINARY`、`NULL`、`UNSIGNED` 和 `ZEROFILL` 数据类型属性。

+   函数和运算符

    +   为了方便从其他 SQL 环境迁移的用户，MySQL Server 支持许多函数的别名。例如，所有字符串函数都支持标准 SQL 语法和 ODBC 语法。

    +   MySQL Server 理解 `||` 和 `&&` 运算符表示逻辑 OR 和 AND，就像 C 编程语言中一样。在 MySQL Server 中，`||` 和 `OR` 是同义词，`&&` 和 `AND` 也是同义词。由于这种良好的语法，MySQL Server 不支持标准 SQL 中用于字符串连接的 `||` 运算符；请使用 `CONCAT()`。由于 `CONCAT()` 接受任意数量的参数，因此很容易将 `||` 运算符的用法转换为 MySQL Server。

    +   在 *`value_list`* 具有多个元素的情况下使用 `COUNT(DISTINCT *`value_list`*)`。

    +   字符串比较默认不区分大小写，排序顺序由当前字符集的排序规则决定，默认为 `utf8mb4`。要执行区分大小写的比较，应该使用 `BINARY` 属性声明列或使用 `BINARY` 转换，这将导致使用基��字符代码值而不是词法排序进行比较。

    +   `%` 运算符是 `MOD()` 的同义词。也就是说，`*`N`* % *`M`*` 等同于 `MOD(*`N`*,*`M`*)`。`%` 支持 C 程序员和与 PostgreSQL 的兼容性。

    +   `=`, `<>`, `<=`, `<`, `>=`, `>`, `<<`, `>>`, `<=>`, `AND`, `OR`, 或 `LIKE` 运算符可以在`SELECT`语句中的输出列列表（`FROM`的左侧）中使用。例如：

        ```sql
        mysql> SELECT col1=1 AND col2=2 FROM my_table;
        ```

    +   `LAST_INSERT_ID()` 函数返回最近的 `AUTO_INCREMENT` 值。参见 Section 14.15, “Information Functions”。

    +   在数字值上允许使用`LIKE`。

    +   `REGEXP` 和 `NOT REGEXP` 扩展正则表达式运算符。

    +   `CONCAT()` 或 `CHAR()` 函数的一个参数或两个以上参数。（在 MySQL Server 中，这些函数可以接受可变数量的参数。）

    +   `BIT_COUNT()`、`CASE`、`ELT()`、`FROM_DAYS()`、`FORMAT()`、`IF()`、`MD5()`、`PERIOD_ADD()`、`PERIOD_DIFF()`、`TO_DAYS()` 和 `WEEKDAY()` 函数。

    +   使用 `TRIM()` 函数修剪子字符串。标准 SQL 仅支持删除单个字符。

    +   `GROUP BY` 函数 `STD()`、`BIT_OR()`、`BIT_AND()`、`BIT_XOR()` 和 `GROUP_CONCAT()`。参见 第 14.19 节，“聚合函数”。
