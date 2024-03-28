# 12.9.2 utf8mb3 字符集（3 字节 UTF-8 Unicode 编码）

> 译文：[`dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf8mb3.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf8mb3.html)

`utf8mb3`字符集具有以下特点：

+   仅支持 BMP 字符（不支持补充字符）

+   每个多字节字符最多需要三个字节。

使用 UTF-8 数据但需要支持补充字符的应用程序应该使用`utf8mb4`而不是`utf8mb3`（参见第 12.9.1 节，“utf8mb4 字符集（4 字节 UTF-8 Unicode 编码）”）。

`utf8mb3`和`ucs2`中提供完全相同的字符集。也就是说，它们具有相同的 repertoire。

注意

MySQL 推荐的字符集是`utf8mb4`。所有新应用程序应该使用`utf8mb4`。

`utf8mb3`字符集已被弃用。`utf8mb3`将在 MySQL 8.0.x 及其后续 LTS 版本系列的生命周期中继续受支持，以及在 MySQL 8.0 中。

预计`utf8mb3`将在 MySQL 的未来主要版本中被移除。

由于更改字符集可能是一个复杂且耗时的任务，您应该立即开始准备使用`utf8mb4`来为新应用程序做好准备。有关转换使用 utfmb3 的现有应用程序的指导，请参见第 12.9.8 节，“在 3 字节和 4 字节 Unicode 字符集之间转换”。

`utf8mb3`可以在`CHARACTER SET`子句中使用，而`utf8mb3_*`collation_substring`*`可以在`COLLATE`子句中使用，其中*`collation_substring`*是`bin`、`czech_ci`、`danish_ci`、`esperanto_ci`、`estonian_ci`等。例如：

```sql
CREATE TABLE t (s1 CHAR(1)) CHARACTER SET utf8mb3;
SELECT * FROM t WHERE s1 COLLATE utf8mb3_general_ci = 'x';
DECLARE x VARCHAR(5) CHARACTER SET utf8mb3 COLLATE utf8mb3_danish_ci;
SELECT CAST('a' AS CHAR CHARACTER SET utf8mb4) COLLATE utf8mb4_czech_ci;
```

在 MySQL 8.0.29 之前，语句中的`utf8mb3`实例会被转换为`utf8`。在 MySQL 8.0.30 及更高版本中，情况相反，因此在`SHOW CREATE TABLE`或`SELECT CHARACTER_SET_NAME FROM INFORMATION_SCHEMA.COLUMNS`或`SELECT COLLATION_NAME FROM INFORMATION_SCHEMA.COLUMNS`等语句中，用户会看到以`utf8mb3`或`utf8mb3_`为前缀的字符集或校对名称。

`utf8mb3`在`CHARACTER SET`子句以外的上下文中也是有效的（但已弃用）。例如：

```sql
mysqld --character-set-server=utf8mb3
```

```sql
SET NAMES 'utf8mb3'; /* and other SET statements that have similar effect */
SELECT _utf8mb3 'a';
```

有关与多字节字符集相关的数据类型存储的信息，请参见字符串类型存储要求。
