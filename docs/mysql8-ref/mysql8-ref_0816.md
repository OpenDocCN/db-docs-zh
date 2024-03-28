# 14.10 转换函数和运算符

> 原文：[`dev.mysql.com/doc/refman/8.0/en/cast-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html)

**表 14.15 转换函数和运算符**

| 名称 | 描述 | 已弃用 |
| --- | --- | --- |
| `BINARY` | 将字符串转换为二进制字符串 | 8.0.27 |
| `CAST()` | 将值转换为特定类型 |  |
| `CONVERT()` | 将值转换为特定类型 |  |

转换函数和运算符使值从一种数据类型转换为另一种数据类型。

+   转换函数和运算符描述

+   字符集转换

+   用于字符串比较的字符集转换

+   空间类型的转换操作

+   转换操作的其他用途

### 转换函数和运算符描述

+   `BINARY` *`expr`*

    `BINARY` 运算符将表达式转换为二进制字符串（具有 `binary` 字符集和 `binary` 校对的字符串）。`BINARY` 的常见用途是通过使用数值字节值而不是逐个字符来进行字符字符串比较。`BINARY` 运算符还导致比较中的尾随空格具有重要意义。有关 `binary` 字符集和非二进制字符集的 `_bin` 校对之间的区别的信息，请参见 第 12.8.5 节，“二进制校对与 _bin 校对的比较”。

    `BINARY` 运算符在 MySQL 8.0.27 中已弃用，并且您应该期望在将来的 MySQL 版本中将其移除。请改用 `CAST(... AS BINARY)`。

    ```sql
    mysql> SET NAMES utf8mb4 COLLATE utf8mb4_general_ci;
     -> OK
    mysql> SELECT 'a' = 'A';
     -> 1
    mysql> SELECT BINARY 'a' = 'A';
     -> 0
    mysql> SELECT 'a' = 'a ';
     -> 1
    mysql> SELECT BINARY 'a' = 'a ';
     -> 0
    ```

    在比较中，`BINARY` 影响整个操作；可以在任一操作数之前使用，结果相同。

    要将字符串表达式转换为二进制字符串，以下结构是等效的：

    ```sql
    CONVERT(*expr* USING BINARY)
    CAST(*expr* AS BINARY)
    BINARY *expr*
    ```

    如果一个值是字符串文字，可以通过使用 `_binary` 字符集引导符将其指定为二进制字符串而无需转换：

    ```sql
    mysql> SELECT 'a' = 'A';
     -> 1
    mysql> SELECT _binary 'a' = 'A';
     -> 0
    ```

    有关引导符的信息，请参见 第 12.3.8 节，“字符集引导符”。

    在表达式中，`BINARY` 运算符的效果与字符列定义中的 `BINARY` 属性不同。对于使用 `BINARY` 属性定义的字符列，MySQL 分配表默认字符集和该字符集的二进制 (`_bin`) 校对。每个非二进制字符集都有一个 `_bin` 校对。例如，如果表默认字符集是 `utf8mb4`，那么这两个列定义是等效的：

    ```sql
    CHAR(10) BINARY
    CHAR(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin
    ```

    在 `CHAR`、`VARCHAR` 或 `TEXT` 列的定义中使用 `CHARACTER SET binary` 会导致该列被视为相应的二进制字符串数据类型。例如，以下定义对是等效的：

    ```sql
    CHAR(10) CHARACTER SET binary
    BINARY(10)

    VARCHAR(10) CHARACTER SET binary
    VARBINARY(10)

    TEXT CHARACTER SET binary
    BLOB
    ```

    如果从 **mysql** 客户端调用 `BINARY`，二进制字符串将使用十六进制表示，具体取决于 `--binary-as-hex` 的值。有关该选项的更多信息，请参阅 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

+   [`CAST(*`expr`* AS *`type`* [ARRAY])`](cast-functions.html#function_cast)

    [`CAST(*`timestamp_value`* AT TIME ZONE *`timezone_specifier`* AS DATETIME[(*`precision`*)])`](cast-functions.html#function_cast)

    *`timezone_specifier`*: [INTERVAL] '+00:00' | 'UTC'

    使用 `CAST(*`expr`* AS *`type`*` 语法，`CAST()` 函数接受任何类型的表达式，并生成指定类型的结果值。这个操作也可以表示为 `CONVERT(*`expr`*, *`type`*)`，它是等效的。如果 *`expr`* 是 `NULL`，`CAST()` 返回 `NULL`。

    允许使用以下 *`type`* 值：

    +   `BINARY[(*`N`*)]`

        生成一个具有 `VARBINARY` 数据类型的字符串，除非表达式 *`expr`* 为空（长度为零），结果类型为 `BINARY(0)`。如果给定可选长度 *`N`*，`BINARY(*`N`*)` 使转换不超过 *`N`* 个字节的参数。长度小于 *`N`* 的值用 `0x00` 字节填充到长度为 *`N`*。如果未给出可选长度 *`N`*，MySQL 从表达式计算最大长度。如果提供或计算的长度大于内部阈值，则结果类型为 `BLOB`。如果长度仍然太长，则结果类型为 `LONGBLOB`。

        有关将 `BINARY` 转换影响比较的描述，请参阅 Section 13.3.3, “The BINARY and VARBINARY Types”。

    +   `CHAR[(*`N`*)] [*`charset_info`*]`

        生成一个具有`VARCHAR`数据类型的字符串，除非表达式 *`expr`* 为空（长度为零），在这种情况下，结果类型为 `CHAR(0)`。如果给定了可选长度 *`N`*，`CHAR(*`N`*)` 使转换不超过 *`N`* 个字符的参数。对于长度小于 *`N`* 的值，不会进行填充。如果未给出可选长度 *`N`*，MySQL 会从表达式计算最大长度。如果提供或计算的长度大于内部阈值，则结果类型为 `TEXT`。如果长度仍然太长，则结果类型为 `LONGTEXT`。

        没有 *`charset_info`* 子句时，`CHAR` 生成一个具有默认字符集的字符串。要明确指定字符集，允许使用以下 *`charset_info`* 值：

        +   `CHARACTER SET *`charset_name`*`：生成具有给定字符集的字符串。

        +   `ASCII`：`CHARACTER SET latin1` 的简写。

        +   `UNICODE`：`CHARACTER SET ucs2` 的简写。

        在所有情况下，字符串具有字符集默认排序规则。

    +   `日期`

        生成一个`日期`值。

    +   `DATETIME[(*`M`*)]`

        生成一个`日期时间`值。如果给定了可选 *`M`* 值，则指定了小数秒精度。

    +   `DECIMAL[(*`M`*[,*`D`*])]`

        生成一个`DECIMAL`值。如果给定了可选的 *`M`* 和 *`D`* 值，则它们指定了数字的最大位数（精度）和小数点后的位数（标度）。如果省略了 *`D`*，则假定为 0。如果省略了 *`M`*，则假定为 10。

    +   `DOUBLE`

        生成一个`DOUBLE`结果。在 MySQL 8.0.17 中添加。

    +   `FLOAT[(*`p`*)]`

        如果未指定精度 *`p`*，则生成一个`FLOAT`类型的结果。如果提供了 *`p`* 并且 0 <= < *`p`* <= 24，则结果为 `FLOAT` 类型。如果 25 <= *`p`* <= 53，则结果为`DOUBLE`类型。如果 *`p`* < 0 或 *`p`* > 53，则返回错误。在 MySQL 8.0.17 中添加。

    +   `JSON`

        生成一个`JSON`值。有关值在`JSON`和其他类型之间转换规则的详细信息，请参阅 JSON 值的比较和排序。

    +   `NCHAR[(*`N`*)]`

        类似于 `CHAR`，但生成具有国家字符集的字符串。请参阅第 12.3.7 节，“国家字符集”。

        与 `CHAR` 不同，`NCHAR` 不允许指定尾随字符集信息。

    +   `REAL`

        产生一个`REAL`类型的结果。如果启用了`REAL_AS_FLOAT` SQL 模式，则实际上是`FLOAT`；否则结果为`DOUBLE`类型。

    +   `SIGNED [INTEGER]`

        产生一个带符号的`BIGINT`值。

    +   *`spatial_type`*

        截至 MySQL 8.0.24 版本，`CAST()` 和 `CONVERT()` 支持将几何值从一种空间类型转换为另一种空间类型，适用于某些空间类型的组合。有关详细信息，请参见空间类型的转换操作。

    +   `TIME[(*`M`*)]`

        产生一个`TIME`值。如果给定了可选的*`M`*值，则指定小数秒精度。

    +   `UNSIGNED [INTEGER]`

        产生一个无符号的`BIGINT`值。

    +   `YEAR`

        产生一个`YEAR`值。在 MySQL 8.0.22 版本中添加。这些规则适用于转换为`YEAR`：

        +   对于范围在 1901-2155（包括）之间的四位数，或者可以解释为此范围内四位数的字符串，返回相应的`YEAR`值。

        +   对于由一到两位数字组成的数字，或者可以解释为这样一个数字的字符串，返回如下`YEAR`值：

            +   如果数字在 1-69（包括）范围内，则加上 2000 并返回总和。

            +   如果数字在 70-99（包括）范围内，则加上 1900 并返回总和。

        +   对于评估为 0 的字符串，返回 2000。

        +   对于数字 0，返回 0。

        +   对于`DATE`、`DATETIME`或`TIMESTAMP`值，返回值的`YEAR`部分。对于`TIME`值，返回当前年份。

            如果不指定`TIME`参数的类型，则可能会得到与预期不同的结果，如下所示：

            ```sql
            mysql> SELECT CAST("11:35:00" AS YEAR), CAST(TIME "11:35:00" AS YEAR);
            +--------------------------+-------------------------------+
            | CAST("11:35:00" AS YEAR) | CAST(TIME "11:35:00" AS YEAR) |
            +--------------------------+-------------------------------+
            |                     2011 |                          2021 |
            +--------------------------+-------------------------------+
            ```

        +   如果参数是`DECIMAL`、`DOUBLE`、`DECIMAL`或`REAL`类型，则将值四舍五入到最接近的整数，然后尝试使用整数值的规则将值转换为`YEAR`，如下所示：

            ```sql
            mysql> SELECT CAST(1944.35 AS YEAR), CAST(1944.50 AS YEAR);
            +-----------------------+-----------------------+
            | CAST(1944.35 AS YEAR) | CAST(1944.50 AS YEAR) |
            +-----------------------+-----------------------+
            |                  1944 |                  1945 |
            +-----------------------+-----------------------+

            mysql> SELECT CAST(66.35 AS YEAR), CAST(66.50 AS YEAR);
            +---------------------+---------------------+
            | CAST(66.35 AS YEAR) | CAST(66.50 AS YEAR) |
            +---------------------+---------------------+
            |                2066 |                2067 |
            +---------------------+---------------------+
            ```

        +   不能将类型为`GEOMETRY`的参数转换为`YEAR`。

        +   对于无法成功转换为`YEAR`的值，返回`NULL`。

        包含必须在转换之前截断的非数字字符的字符串值会引发警告，如下所示：

        ```sql
        mysql> SELECT CAST("1979aaa" AS YEAR);
        +-------------------------+
        | CAST("1979aaa" AS YEAR) |
        +-------------------------+
        |                    1979 |
        +-------------------------+
        1 row in set, 1 warning (0.00 sec)

        mysql> SHOW WARNINGS;
        +---------+------+-------------------------------------------+
        | Level   | Code | Message                                   |
        +---------+------+-------------------------------------------+
        | Warning | 1292 | Truncated incorrect YEAR value: '1979aaa' |
        +---------+------+-------------------------------------------+
        ```

    在 MySQL 8.0.17 及更高版本中，`InnoDB`允许在`CREATE INDEX`、`CREATE TABLE`和`ALTER TABLE`语句中使用额外的`ARRAY`关键字来创建 JSON 数组的多值索引。除了在这些语句中用于创建多值索引时，不支持`ARRAY`。被索引的列必须是 JSON 类型的列。使用`ARRAY`时，`AS`关键字后面的*`type`*可以指定`CAST()`支持的任何类型，但不包括`BINARY`、`JSON`和`YEAR`。有关语法信息和示例，以及其他相关信息，请参见多值索引。

    注意

    与`CAST()`不同，`CONVERT()` *不*支持多值索引创建或`ARRAY`关键字。

    从 MySQL 8.0.22 开始，`CAST()`支持使用`AT TIMEZONE`运算符将`TIMESTAMP`值检索为 UTC 时间。唯一支持的时区是 UTC；可以将其指定为`'+00:00'`或`'UTC'`。此语法支持的唯一返回类型是`DATETIME`，其精度范围为 0 到 6，包括 0 和 6。

    使用时区偏移的`TIMESTAMP`值也受支持。

    ```sql
    mysql> SELECT @@system_time_zone;
    +--------------------+
    | @@system_time_zone |
    +--------------------+
    | EDT                |
    +--------------------+
    1 row in set (0.00 sec)

    mysql> CREATE TABLE TZ (c TIMESTAMP);
    Query OK, 0 rows affected (0.41 sec)

    mysql> INSERT INTO tz VALUES
     ->     ROW(CURRENT_TIMESTAMP),
     ->     ROW('2020-07-28 14:50:15+1:00');
    Query OK, 1 row affected (0.08 sec)

    mysql> TABLE tz;
    +---------------------+
    | c                   |
    +---------------------+
    | 2020-07-28 09:22:41 |
    | 2020-07-28 09:50:15 |
    +---------------------+
    2 rows in set (0.00 sec)

    mysql> SELECT CAST(c AT TIME ZONE '+00:00' AS DATETIME) AS u FROM tz;
    +---------------------+
    | u                   |
    +---------------------+
    | 2020-07-28 13:22:41 |
    | 2020-07-28 13:50:15 |
    +---------------------+
    2 rows in set (0.00 sec)

    mysql> SELECT CAST(c AT TIME ZONE 'UTC' AS DATETIME(2)) AS u FROM tz;
    +------------------------+
    | u                      |
    +------------------------+
    | 2020-07-28 13:22:41.00 |
    | 2020-07-28 13:50:15.00 |
    +------------------------+
    2 rows in set (0.00 sec)
    ```

    如果您在`CAST()`的这种形式中使用`'UTC'`作为时区指定符，并且服务器引发错误，例如未知或不正确的时区：'UTC'，则可能需要安装 MySQL 时区表（请参阅填充时区表）。

    `AT TIME ZONE`不支持`ARRAY`关键字，并且不受`CONVERT()`函数支持。

+   `CONVERT(*`expr`* USING *`transcoding_name`*)`

    `CONVERT(*`expr`*,*`type`*)`

    `CONVERT(*`expr`* USING *`transcoding_name`*)`是标准的 SQL 语法。`CONVERT()`的非`USING`形式是 ODBC 语法。无论使用哪种语法，如果*`expr`*为`NULL`，该函数都会返回`NULL`。

    `CONVERT(*`expr`* USING *`transcoding_name`*)`在不同字符集之间转换数据。在 MySQL 中，转码名称与相应的字符集名称相同。例如，此语句将默认字符集中的字符串`'abc'`转换为`utf8mb4`字符集中的相应字符串：

    ```sql
    SELECT CONVERT('abc' USING utf8mb4);
    ```

    `CONVERT(*`expr`*, *`type`*)`语法（不带`USING`）接受一个表达式和一个*`type`*值，指定结果类型，并产生指定类型的结果值。此操作也可以表示为`CAST(*`expr`* AS *`type`*)`，这是等效的。有关更多信息，请参阅`CAST()`的描述。

    注意

    在 MySQL 8.0.28 之前，这个函数有时允许将`BINARY`值无效地转换为非二进制字符集。当`CONVERT()`作为索引生成列表达式的一部分时，这可能导致在从先前版本的 MySQL 升级后出现索引损坏。有关如何处理这种情况的信息，请参见 SQL 更改。

### 字符集转换

带有`USING`子句的`CONVERT()`在字符集之间转换数据：

```sql
CONVERT(*expr* USING *transcoding_name*)
```

在 MySQL 中，转码名称与相应的字符集名称相同。

例子：

```sql
SELECT CONVERT('test' USING utf8mb4);
SELECT CONVERT(_latin1'Müller' USING utf8mb4);
INSERT INTO utf8mb4_table (utf8mb4_column)
    SELECT CONVERT(latin1_column USING utf8mb4) FROM latin1_table;
```

要在字符集之间转换字符串，还可以使用`CONVERT(*`expr`*, *`type`*)`语法（不带`USING`），或者等效的`CAST(*`expr`* AS *`type`*)`：

```sql
CONVERT(*string*, CHAR[(*N*)] CHARACTER SET *charset_name*)
CAST(*string* AS CHAR[(*N*)] CHARACTER SET *charset_name*)
```

例子：

```sql
SELECT CONVERT('test', CHAR CHARACTER SET utf8mb4);
SELECT CAST('test' AS CHAR CHARACTER SET utf8mb4);
```

如果你像刚才展示的那样指定`CHARACTER SET *`charset_name`*`，结果的字符集和排序规则分别为*`charset_name`*和*`charset_name`*的默认排序规则。如果省略`CHARACTER SET *`charset_name`*`，结果的字符集和排序规则由确定默认连接字符集和排序规则的`character_set_connection`和`collation_connection`系统变量定义（参见第 12.4 节，“连接字符集和排序规则”）。

在`CONVERT()`或`CAST()`调用中不允许使用`COLLATE`子句，但可以将其应用于函数结果。例如，以下是合法的：

```sql
SELECT CONVERT('test' USING utf8mb4) COLLATE utf8mb4_bin;
SELECT CONVERT('test', CHAR CHARACTER SET utf8mb4) COLLATE utf8mb4_bin;
SELECT CAST('test' AS CHAR CHARACTER SET utf8mb4) COLLATE utf8mb4_bin;
```

但以下是不合法的：

```sql
SELECT CONVERT('test' USING utf8mb4 COLLATE utf8mb4_bin);
SELECT CONVERT('test', CHAR CHARACTER SET utf8mb4 COLLATE utf8mb4_bin);
SELECT CAST('test' AS CHAR CHARACTER SET utf8mb4 COLLATE utf8mb4_bin);
```

对于字符串字面值，指定字符集的另一种方法是使用字符集引导符。在前面的示例中，`_latin1`和`_latin2`是引导符的实例。与转换函数（如`CAST()`或`CONVERT()`）不同，这些函数将字符串从一个字符集转换为另一个字符集，引导符指定字符串字面值具有特定的字符集，不涉及转换。有关更多信息，请参见第 12.3.8 节，“字符集引导符”。

### 字符串比较的字符集转换

通常情况下，你无法以不区分大小写的方式比较`BLOB`值或其他二进制字符串，因为二进制字符串使用`binary`字符集，该字符集没有与大小写概念相关的排序规则。要执行不区分大小写的比较，首先使用`CONVERT()`或`CAST()`函数将值转换为非二进制字符串。对转换后的字符串进行比较时使用其排序规则。例如，如果转换结果的排序规则不区分大小写，则`LIKE`操作也不区分大小写。这对于以下操作是正确的，因为默认的`utf8mb4`排序规则（`utf8mb4_0900_ai_ci`）不区分大小写：

```sql
SELECT 'A' LIKE CONVERT(*blob_col* USING utf8mb4)
  FROM *tbl_name*;
```

要为转换后的字符串指定特定的排序规则，请在`CONVERT()`调用后使用`COLLATE`子句：

```sql
SELECT 'A' LIKE CONVERT(*blob_col* USING utf8mb4) COLLATE utf8mb4_unicode_ci
  FROM *tbl_name*;
```

要使用不同的字符集，请在前述语句中用其名称替换`utf8mb4`（类似地，要使用不同的排序规则也是如此）。

`CONVERT()`和`CAST()`可以更普遍地用于比较以不同字符集表示的字符串。例如，对这些字符串进行比较会导致错误，因为它们具有不同的字符集：

```sql
mysql> SET @s1 = _latin1 'abc', @s2 = _latin2 'abc';
mysql> SELECT @s1 = @s2;
ERROR 1267 (HY000): Illegal mix of collations (latin1_swedish_ci,IMPLICIT)
and (latin2_general_ci,IMPLICIT) for operation '='
```

将其中一个字符串转换为与另一个兼容的字符集，使比较可以顺利进行：

```sql
mysql> SELECT @s1 = CONVERT(@s2 USING latin1);
+---------------------------------+
| @s1 = CONVERT(@s2 USING latin1) |
+---------------------------------+
|                               1 |
+---------------------------------+
```

字符集转换在对二进制字符串进行大小写转换之前也很有用。当直接应用于二进制字符串时，`LOWER()`和`UPPER()`是无效的，因为大小写概念不适用。要对二进制字符串执行大小写转换，首先使用适合字符串中存储的数据的字符集将其转换为非二进制字符串：

```sql
mysql> SET @str = BINARY 'New York';
mysql> SELECT LOWER(@str), LOWER(CONVERT(@str USING utf8mb4));
+-------------+------------------------------------+
| LOWER(@str) | LOWER(CONVERT(@str USING utf8mb4)) |
+-------------+------------------------------------+
| New York    | new york                           |
+-------------+------------------------------------+
```

请注意，如果将`BINARY`、`CAST()`或`CONVERT()`应用于索引列，MySQL 可能无法有效地使用索引。

### 对空间类型执行转换操作

从 MySQL 8.0.24 开始，`CAST()`和`CONVERT()`支持将几何值从一种空间类型转换为另一种空间类型，适用于某些空间类型的组合。以下列表显示了允许的类型组合，其中“MySQL 扩展”指的是 MySQL 中实现的超出 SQL/MM 标准定义的转换：

+   从`Point`到：

    +   `MultiPoint`

    +   `GeometryCollection`

+   从`LineString`到：

    +   `Polygon`（MySQL 扩展）

    +   `MultiPoint`（MySQL 扩展）

    +   `MultiLineString`

    +   `GeometryCollection`

+   从`Polygon`到：

    +   `LineString`（MySQL 扩展）

    +   `MultiLineString`（MySQL 扩展）

    +   `MultiPolygon`

    +   `GeometryCollection`

+   从`MultiPoint`到：

    +   `Point`

    +   `LineString`（MySQL 扩展）

    +   `GeometryCollection`

+   从`MultiLineString`到：

    +   `LineString`

    +   `Polygon`（MySQL 扩展）

    +   `MultiPolygon`（MySQL 扩展）

    +   `GeometryCollection`

+   从`MultiPolygon`到：

    +   `Polygon`

    +   `MultiLineString`（MySQL 扩展）

    +   `GeometryCollection`

+   从`GeometryCollection`到：

    +   `Point`

    +   `LineString`

    +   `Polygon`

    +   `MultiPoint`

    +   `MultiLineString`

    +   `MultiPolygon`

在空间转换中，`GeometryCollection`和`GeomCollection`是相同结果类型的同义词。

一些条件适用于所有空间类型转换，而一些条件仅适用于转换结果为特定空间类型时。有关“格式良好的几何”等术语的信息，请参阅第 13.4.4 节，“几何格式良好性和有效性”。

+   空间转换的一般条件

+   转换为 Point 的条件

+   转换为 LineString 的条件

+   转换为 Polygon 的条件

+   转换为 MultiPoint 的条件

+   转换为 MultiLineString 的条件

+   转换为 MultiPolygon 的条件

+   转换为 GeometryCollection 的条件

#### 空间转换的一般条件

这些条件适用于所有空间转换，无论结果类型如何：

+   转换的结果与要转换的表达式的 SRS 相同。

+   在空间类型之间进行转换不会改变坐标值或顺序。

+   如果要转换的表达式为`NULL`，函数结果为`NULL`。

+   不允许使用带有指定空间类型的`RETURNING`子句的`JSON_VALUE()`函数来转换空间类型。

+   不允许将空间类型转换为`ARRAY`。

+   如果空间类型组合是允许的，但要转换的表达式不是一个语法上良好形成的几何图形，将会发生一个`ER_GIS_INVALID_DATA`错误。

+   如果空间类型组合是允许的，但要转换的表达式是在未定义的空间参考系统（SRS）中语法上良好形成的几何图形，则会发生一个`ER_SRS_NOT_FOUND`错误。

+   如果要转换的表达式具有地理 SRS 但经度或纬度超出范围，则会发生错误：

    +   如果经度值不在范围（−180, 180]内，则会发生一个`ER_GEOMETRY_PARAM_LONGITUDE_OUT_OF_RANGE`错误。

    +   如果纬度值不在范围[−90, 90]内，则会发生一个`ER_GEOMETRY_PARAM_LATITUDE_OUT_OF_RANGE`错误。

    显示的范围以度为单位。如果 SRS 使用另一个单位，则范围使用其单位中的相应值。由于浮点运算，确切的范围限制略有偏差。

#### 转换为点的条件

当转换结果类型为`Point`时，应用以下条件：

+   如果要转换的表达式是一个`Point`类型的良好形成的几何图形，则函数结果为该`Point`。

+   如果要转换的表达式是一个只包含单个`Point`的`MultiPoint`类型的良好形成的几何图形，则函数结果为该`Point`。如果表达式包含多个`Point`，则会发生一个`ER_INVALID_CAST_TO_GEOMETRY`错误。

+   如果要转换的表达式是一个只包含单个`Point`的`GeometryCollection`类型的良好形成的几何图形，则函数结果为该`Point`。如果表达式为空，包含多个`Point`或包含其他几何类型，则会发生一个`ER_INVALID_CAST_TO_GEOMETRY`错误。

+   如果要转换的表达式是除`Point`、`MultiPoint`、`GeometryCollection`之外的其他类型的良好形成的几何图形，则会发生一个`ER_INVALID_CAST_TO_GEOMETRY`错误。

#### 转换为 LineString 的条件

当转换结果类型为`LineString`时，应用以下条件：

+   如果要转换的表达式是一个`LineString`类型的良好形成的几何图形，则函数结果为该`LineString`。

+   如果要转换的表达式是类型为`Polygon`且没有内环的形状，则函数结果是一个`LineString`，其中包含外环的点按相同顺序排列。如果表达式有内环，则会出现`ER_INVALID_CAST_TO_GEOMETRY`错误。

+   如果要转换的表达式是包含至少两个点的类型为`MultiPoint`的形状，则函数结果是一个`LineString`，其中包含`MultiPoint`中点的顺序与表达式中出现的顺序相同。如果表达式只包含一个`Point`，则会出现`ER_INVALID_CAST_TO_GEOMETRY`错误。

+   如果要转换的表达式是包含单个`LineString`的类型为`MultiLineString`的形状，则函数结果是该`LineString`。如果表达式包含多个`LineString`，则会出现`ER_INVALID_CAST_TO_GEOMETRY`错误。

+   如果要转换的表达式是包含单个`LineString`的类型为`GeometryCollection`的形状，则函数结果是该`LineString`。如果表达式为空，包含多个`LineString`或包含其他几何类型，则会出现`ER_INVALID_CAST_TO_GEOMETRY`错误。

+   如果要转换的表达式是除`LineString`、`Polygon`、`MultiPoint`、`MultiLineString`或`GeometryCollection`之外类型的形状，则会出现`ER_INVALID_CAST_TO_GEOMETRY`错误。

#### 转换为多边形的条件

当转换结果类型为`Polygon`时，应满足以下条件：

+   如果要转换的表达式是类型为`LineString`的形状，且是一个环（即起点和终点相同），则函数结果是一个外环由`LineString`中的点按相同顺序组成的`Polygon`。如果表达式不是环，则会出现`ER_INVALID_CAST_TO_GEOMETRY`错误。如果环的顺序不正确（外环必须是逆时针方向），则会出现`ER_INVALID_CAST_POLYGON_RING_DIRECTION`错误。

+   如果要转换的表达式是类型为`Polygon`的形状，则函数结果是该`Polygon`。

+   如果要转换的表达式是类型为`MultiLineString`且所有元素都是环的形状良好的几何图形，则函数结果是以第一个`LineString`作为外环，任何额外的`LineString`值作为内环的`Polygon`。如果表达式的任何元素不是环，则会发生`ER_INVALID_CAST_TO_GEOMETRY`错误。如果任何环不按正确顺序排列（外环必须是逆时针，内环必须是顺时针），则会发生`ER_INVALID_CAST_POLYGON_RING_DIRECTION`错误。

+   如果要转换的表达式是类型为`MultiPolygon`且包含单个`Polygon`的形状良好的几何图形，则函数结果是该`Polygon`。如果表达式包含多个`Polygon`，则会发生`ER_INVALID_CAST_TO_GEOMETRY`错误。

+   如果要转换的表达式是类型为`GeometryCollection`且仅包含单个`Polygon`的形状良好的几何图形，则函数结果是该`Polygon`。如果表达式为空，包含多个`Polygon`或包含其他几何类型，则会发生`ER_INVALID_CAST_TO_GEOMETRY`错误。

+   如果要转换的表达式是类型为`LineString`、`Polygon`、`MultiLineString`、`MultiPolygon`或`GeometryCollection`之外的形状良好的几何图形，则会发生`ER_INVALID_CAST_TO_GEOMETRY`错误。

#### 转换为 MultiPoint 的条件

当转换结果类型为`MultiPoint`时，应满足以下条件：

+   如果要转换的表达式是类型为`Point`的形状良好的几何图形，则函数结果是包含该`Point`作为唯一元素的`MultiPoint`。

+   如果要转换的表达式是类型为`LineString`的形状良好的几何图形，则函数结果是按相同顺序包含`LineString`的点的`MultiPoint`。

+   如果要转换的表达式是类型为`MultiPoint`的形状良好的几何图形，则函数结果是该`MultiPoint`。

+   如果要转换的表达式是类型为`GeometryCollection`且仅包含点的形状良好的几何图形，则函数结果是包含这些点的`MultiPoint`。如果`GeometryCollection`为空或包含其他几何类型，则会发生`ER_INVALID_CAST_TO_GEOMETRY`错误。

+   如果要转换的表达式是类型为`Point`、`LineString`、`MultiPoint`或`GeometryCollection`之外的形状良好的几何图形，则会发生`ER_INVALID_CAST_TO_GEOMETRY`错误。

#### 转换为 MultiLineString 的条件

当转换结果类型为`MultiLineString`时，应满足以下条件：

+   如果要转换的表达式是类型为`LineString`的格式良好的几何图形，则函数结果为包含该`LineString`作为唯一元素的`MultiLineString`。

+   如果要转换的表达式是类型为`Polygon`的格式良好的几何图形，则函数结果为包含`Polygon`的外环作为第一个元素，任何内环作为额外元素按表达式中出现顺序排列的`MultiLineString`。

+   如果要转换的表达式是类型为`MultiLineString`的格式良好的几何图形，则函数结果为该`MultiLineString`。

+   如果要转换的表达式是仅包含没有内环的多边形的`MultiPolygon`的格式良好的几何图形，则函数结果为包含多边形环按表达式中出现顺序排列的`MultiLineString`。如果表达式包含任何带有内环的多边形，则会发生`ER_WRONG_PARAMETERS_TO_STORED_FCT`错误。

+   如果要转换的表达式是仅包含线串的`GeometryCollection`的格式良好的几何图形，则函数结果为包含这些线串的`MultiLineString`。如果表达式为空或包含其他几何类型，则会发生`ER_INVALID_CAST_TO_GEOMETRY`错误。

+   如果要转换的表达式是类型为`LineString`、`Polygon`、`MultiLineString`、`MultiPolygon`或`GeometryCollection`之外的格式良好的几何图形，则会发生`ER_INVALID_CAST_TO_GEOMETRY`错误。

#### 转换为 MultiPolygon 的条件

当转换结果类型为`MultiPolygon`时，应满足以下条件：

+   如果要转换的表达式是类型为`Polygon`的格式良好的几何图形，则函数结果为包含该`Polygon`作为唯一元素的`MultiPolygon`。

+   如果要转换的表达式是所有元素均为环的类型为`MultiLineString`的格式良好的几何图形，则函数结果为包含每个元素的仅具有外环的`Polygon`的`MultiPolygon`。如果任何元素不是环，则会发生`ER_INVALID_CAST_TO_GEOMETRY`错误。如果任何环不按正确顺序（外环必须逆时针）排列，则会发生`ER_INVALID_CAST_POLYGON_RING_DIRECTION`错误。

+   如果要转换的表达式是类型为`MultiPolygon`的格式良好的几何图形，则函数结果为该`MultiPolygon`。

+   如果要转换的表达式是一个只包含多边形的`GeometryCollection`类型的形状，函数的结果是包含这些多边形的`MultiPolygon`。如果表达式为空或包含其他几何类型，则会出现`ER_INVALID_CAST_TO_GEOMETRY`错误。

+   如果要转换的表达式是除了`Polygon`、`MultiLineString`、`MultiPolygon`或`GeometryCollection`之外的其他类型的形状，会出现`ER_INVALID_CAST_TO_GEOMETRY`错误。

#### 转换为 GeometryCollection 的条件

当转换结果类型为`GeometryCollection`时，这些条件适用：

+   `GeometryCollection`和`GeomCollection`是相同结果类型的同义词。

+   如果要转换的表达式是一个只包含点的`Point`类型的形状，函数的结果是包含该点作为唯一元素的`GeometryCollection`。

+   如果要转换的表达式是一个只包含线串的`LineString`类型的形状，函数的结果是包含该线串作为唯一元素的`GeometryCollection`。

+   如果要转换的表达式是一个只包含多边形的`Polygon`类型的形状，函数的结果是包含该多边形作为唯一元素的`GeometryCollection`。

+   如果要转换的表达式是一个只包含多点的`MultiPoint`类型的形状，函数的结果是按照表达式中出现的顺序包含这些点的`GeometryCollection`。

+   如果要转换的表达式是一个只包含多线串的`MultiLineString`类型的形状，函数的结果是按照表达式中出现的顺序包含这些线串的`GeometryCollection`。

+   如果要转换的表达式是一个只包含多边形的`MultiPolygon`类型的形状，函数的结果是按照表达式中出现的顺序包含`MultiPolygon`的元素的`GeometryCollection`。

+   如果要转换的表达式是一个只包含 GeometryCollection 的`GeometryCollection`类型的形状，函数的结果就是那个`GeometryCollection`。

### 转换操作的其他用途

转换函数在`CREATE TABLE ... SELECT`语句中创建具有特定类型的列时非常有用：

```sql
mysql> CREATE TABLE new_table SELECT CAST('2000-01-01' AS DATE) AS c1;
mysql> SHOW CREATE TABLE new_table\G
*************************** 1\. row ***************************
       Table: new_table
Create Table: CREATE TABLE `new_table` (
  `c1` date DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

转换函数对于按字典顺序排序`ENUM`列非常有用。通常，对`ENUM`列的排序使用内部数值。将值转换为`CHAR`会导致字典排序：

```sql
SELECT *enum_col* FROM *tbl_name* ORDER BY CAST(*enum_col* AS CHAR);
```

`CAST()`也会改变结果，如果你将其作为更复杂表达式的一部分，比如`CONCAT('Date: ',CAST(NOW() AS DATE))`。

对于时间值，很少需要使用`CAST()`以不同格式提取数据。而是使用诸如`EXTRACT()`、`DATE_FORMAT()`或`TIME_FORMAT()`等函数。请参阅第 14.7 节，“日期和时间函数”。

要将字符串转换为数字，通常只需在数字上下文中使用字符串值：

```sql
mysql> SELECT 1+'1';
 -> 2
```

十六进制和位字面值也是默认的二进制字符串。

```sql
mysql> SELECT X'41', X'41'+0;
 -> 'A', 65
mysql> SELECT b'1100001', b'1100001'+0;
 -> 'a', 97
```

在算术运算中使用的字符串在表达式评估期间被转换为浮点数。

在字符串上下文中使用的数字被转换为字符串：

```sql
mysql> SELECT CONCAT('hello you ',2);
 -> 'hello you 2'
```

有关将数字隐式转换为字符串的信息，请参阅第 14.3 节，“表达式评估中的类型转换”。

MySQL 支持带符号和无符号 64 位值的算术运算。对于数值运算符（如`+`或`-`），其中一个操作数是无符号整数时，默认结果是无符号的（参见第 14.6.1 节，“算术运算符”）。要覆盖此行为，使用`SIGNED`或`UNSIGNED`强制转换运算符将值分别转换为有符号或无符号 64 位整数。

```sql
mysql> SELECT 1 - 2;
 -> -1
mysql> SELECT CAST(1 - 2 AS UNSIGNED);
 -> 18446744073709551615
mysql> SELECT CAST(CAST(1 - 2 AS UNSIGNED) AS SIGNED);
 -> -1
```

如果任一操作数是浮点值，则结果是浮点值，并且不受前述规则的影响。（在此上下文中，`DECIMAL`列值被视为浮点值。）

```sql
mysql> SELECT CAST(1 AS UNSIGNED) - 2.0;
 -> -1.0
```

SQL 模式会影响转换操作的结果（请参阅第 7.1.11 节，“服务器 SQL 模式”）。示例：

+   对于将“零”日期字符串转换为日期，当启用`NO_ZERO_DATE` SQL 模式时，`CONVERT()`和`CAST()`返回`NULL`并产生警告。

+   对于整数减法，如果启用了`NO_UNSIGNED_SUBTRACTION` SQL 模式，则减法结果是有符号的，即使任何操作数都是无符号的。
