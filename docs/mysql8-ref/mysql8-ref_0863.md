# 14.19.1 聚合函数描述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html)

本节描述了对值集合进行操作的聚合函数。它们通常与`GROUP BY`子句一起使用，将值分组为子集。

**表 14.29 聚合函数**

| 名称 | 描述 |
| --- | --- |
| `AVG()` | 返回参数的平均值 |
| `BIT_AND()` | 返回按位与 |
| `BIT_OR()` | 返回按位或 |
| `BIT_XOR()` | 返回按位异或 |
| `COUNT()` | 返回返回的行数计数 |
| `COUNT(DISTINCT)` | 返回不同值的数量 |
| `GROUP_CONCAT()` | 返回连接的字符串 |
| `JSON_ARRAYAGG()` | 将结果集作为单个 JSON 数组返回 |
| `JSON_OBJECTAGG()` | 将结果集作为单个 JSON 对象返回 |
| `MAX()` | 返回最大值 |
| `MIN()` | 返回最小值 |
| `STD()` | 返回总体标准偏差 |
| `STDDEV()` | 返回总体标准偏差 |
| `STDDEV_POP()` | 返回总体标准偏差 |
| `STDDEV_SAMP()` | 返回样本标准偏差 |
| `SUM()` | 返回总和 |
| `VAR_POP()` | 返回总体标准方差 |
| `VAR_SAMP()` | 返回样本方差 |
| `VARIANCE()` | 返回总体标准方差 |
| 名称 | 描述 |

除非另有说明，聚合函数会忽略`NULL`值。

如果在不包含`GROUP BY`子句的语句中使用聚合函数，则相当于对所有行进行分组。有关更多信息，请参见第 14.19.3 节，“MySQL GROUP BY 的处理”。

大多数聚合函数都可以作为窗口函数使用。可以这样使用的函数在其语法描述中通过`[*`over_clause`*]`表示，表示一个可选的`OVER`子句。*`over_clause`*在第 14.20.2 节，“窗口函数概念和语法”中描述，该节还包括有关窗口函数使用的其他信息。

对于数值参数，方差和标准差函数返回一个`DOUBLE`值。`SUM()`和`AVG()`函数对于精确值参数（整数或`DECIMAL`）返回一个`DECIMAL`值，对于近似值参数（`FLOAT`或`DOUBLE`）返回一个`DOUBLE`值。

`SUM()`和`AVG()`聚合函数不适用于时间值。（它们将值转换为数字，丢失第一个非数字字符后的所有内容。）为解决此问题，将值转换为数字单位，执行聚合操作，然后将其转换回时间值。示例：

```sql
SELECT SEC_TO_TIME(SUM(TIME_TO_SEC(*time_col*))) FROM *tbl_name*;
SELECT FROM_DAYS(SUM(TO_DAYS(*date_col*))) FROM *tbl_name*;
```

诸如`SUM()`或`AVG()`这样期望数值参数的函数，如有必要会将参数转换为数字。对于`SET`或`ENUM`值，转换操作会使用底层的数值。

`BIT_AND()`、`BIT_OR()`和`BIT_XOR()`聚合函数执行位操作。在 MySQL 8.0 之前，位函数和运算符需要`BIGINT`（64 位整数）参数，并返回`BIGINT`值，因此它们的最大范围为 64 位。非`BIGINT`参数在执行操作之前被转换为`BIGINT`，可能会发生截断。

在 MySQL 8.0 中，位函数和运算符允许二进制字符串类型参数（`BINARY`、`VARBINARY`和`BLOB`类型），并返回相同类型的值，这使它们能够接受参数并生成大于 64 位的返回值。有关位操作的参数评估和结果类型的讨论，请参见第 14.12 节“位函数和运算符”中的介绍性讨论。

+   [`AVG([DISTINCT] *`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_avg)

    返回`*`expr`*`的平均值。`DISTINCT`选项可用于返回*`expr`*的不同值的平均值。

    如果没有匹配的行，`AVG()`返回`NULL`。如果*`expr`*为`NULL`，该函数也返回`NULL`。

    如果*`over_clause`*存在，则此函数作为窗口函数执行。*`over_clause`*如第 14.20.2 节“窗口函数概念和语法”中所述；它不能与`DISTINCT`一起使用。

    ```sql
    mysql> SELECT student_name, AVG(test_score)
           FROM student
           GROUP BY student_name;
    ```

+   [`BIT_AND(*`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_bit-and)

    返回*`expr`*中所有位的按位`AND`。

    结果类型取决于函数参数值是作为二进制字符串还是数字进行评估：

    +   当参数值具有二进制字符串类型且参数不是十六进制文字、位文字或`NULL`文字时，进行二进制字符串评估。否则进行数值评估，必要时将参数值转换为无符号 64 位整数。

    +   二进制字符串评估会产生与参数值相同长度的二进制字符串。如果参数值长度不相等，则会出现`ER_INVALID_BITWISE_OPERANDS_SIZE`错误。如果参数大小超过 511 字节，则会出现`ER_INVALID_BITWISE_AGGREGATE_OPERANDS_SIZE`错误。数值评估会产生一个无符号 64 位整数。

    如果没有匹配的行，`BIT_AND()`返回一个中性值（所有位设置为 1），其长度与参数值相同。

    `NULL`值不会影响结果，除非所有值都是`NULL`。在这种情况下，结果是一个中性值，其长度与参数值相同。

    有关参数评估和结果类型的更多信息，请参见第 14.12 节，“位函数和运算符”中的介绍性讨论。

    如果从**mysql**客户端中调用`BIT_AND()`，二进制字符串结果将根据`--binary-as-hex`的值使用十六进制表示。有关该选项的更多信息，请参见第 6.5.1 节，“mysql — MySQL 命令行客户端”。

    从 MySQL 8.0.12 开始，如果存在*`over_clause`*，此函数将作为窗口函数执行。*`over_clause`*的描述如第 14.20.2 节，“窗口函数概念和语法”中所述。

+   [`BIT_OR(*`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_bit-or)

    ��回*`expr`*中所有位的按位`OR`。

    结果类型取决于函数参数值是作为二进制字符串还是数字进行评估：

    +   当参数值具有二进制字符串类型且参数不是十六进制文字、位文字或`NULL`文字时，会发生二进制字符串评估。否则会发生数值评估，必要时将参数值转换为无符号 64 位整数。

    +   二进制字符串评估会产生与参数值相同长度的二进制字符串。如果参数值长度不相等，则会出现`ER_INVALID_BITWISE_OPERANDS_SIZE`错误。如果参数大小超过 511 字节，则会出现`ER_INVALID_BITWISE_AGGREGATE_OPERANDS_SIZE`错误。数值评估会产生一个无符号 64 位整数。

    如果没有匹配的行，`BIT_OR()` 返回一个中性值（所有位设置为 0），其长度与参数值相同。

    `NULL` 值不会影响结果，除非所有值都是 `NULL`。在这种情况下，结果是一个中性值，其长度与参数值相同。

    有关参数评估和结果类型的更多信息，请参见 Section 14.12, “Bit Functions and Operators” 中的介绍性讨论。

    如果从 **mysql** 客户端内调用 `BIT_OR()`，二进制字符串结果将根据 `--binary-as-hex` 的值以十六进制表示。有关该选项的更多信息，请参见 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

    从 MySQL 8.0.12 开始，如果存在 *`over_clause`*，此函数将作为窗口函数执行。*`over_clause`* 如 Section 14.20.2, “Window Function Concepts and Syntax” 中所述。

+   [`BIT_XOR(*`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_bit-xor)

    返回 *`expr`* 中所有位的按位 `XOR`。

    结果类型取决于函数参数值是作为二进制字符串还是���字进行评估：

    +   当参数值具有二进制字符串类型且参数不是十六进制文字、位文字或 `NULL` 文字时，进行二进制字符串评估。否则进行数字评估，必要时将参数值转换为无符号 64 位整数。

    +   二进制字符串评估产生与参数值相同长度的二进制字符串。如果参数值长度不相等，则会出现 `ER_INVALID_BITWISE_OPERANDS_SIZE` 错误。如果参数大小超过 511 字节，则会出现 `ER_INVALID_BITWISE_AGGREGATE_OPERANDS_SIZE` 错误。数字评估产生一个无符号 64 位整数。

    如果没有匹配的行，`BIT_XOR()` 返回一个中性值（所有位设置为 0），其长度与参数值相同。

    `NULL` 值不会影响结果，除非所有值都是 `NULL`。在这种情况下，结果是一个中性值，其长度与参数值相同。

    有关参数评估和结果类型的更多信息，请参见 Section 14.12, “Bit Functions and Operators” 中的介绍性讨论。

    如果从 **mysql** 客户端调用 `BIT_XOR()`，二进制字符串结果将使用十六进制表示，取决于 `--binary-as-hex` 的值。有关该选项的更多信息，请参见 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

    截至 MySQL 8.0.12，如果存在 *`over_clause`*，此函数将作为窗口函数执行。*`over_clause`* 如 Section 14.20.2, “Window Function Concepts and Syntax” 中所述。

+   [`COUNT(*`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_count)

    返回 `SELECT` 语句检索的行中 *`expr`* 的非 `NULL` 值的计数。结果是一个 `BIGINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT") 值。

    如果没有匹配的行，`COUNT()` 返回 `0`。`COUNT(NULL)` 返回 0。

    如果存在 *`over_clause`*，此函数将作为窗口函数执行。*`over_clause`* 如 Section 14.20.2, “Window Function Concepts and Syntax” 中所述。

    ```sql
    mysql> SELECT student.student_name,COUNT(*)
           FROM student,course
           WHERE student.student_id=course.student_id
           GROUP BY student_name;
    ```

    `COUNT(*)` 有些不同，它返回检索到的行数，无论它们是否包含 `NULL` 值。

    对于像 `InnoDB` 这样的事务性存储引擎，存储精确的行计数是有问题的。可能同时发生多个事务，每个事务可能会影响计数。

    `InnoDB` 不会保留表中行的内部计数，因为并发事务可能同时“看到”不同数量的行。因此，`SELECT COUNT(*)` 语句仅计算当前事务可见的行数。

    截至 MySQL 8.0.13，对于 `InnoDB` 表，`SELECT COUNT(*) FROM *`tbl_name`*` 查询性能在没有额外子句（如 `WHERE` 或 `GROUP BY`）的情况下针对单线程工作负载进行了优化。

    `InnoDB` 通过遍历最小可用的次要索引来处理 `SELECT COUNT(*)` 语句，除非索引或优化器提示指示优化器使用不同的索引。如果不存在次要索引，则 `InnoDB` 通过扫描聚簇索引来处理 `SELECT COUNT(*)` 语句。

    处理 `SELECT COUNT(*)` 语句需要一些时间，如果索引记录不完全在缓冲池中。为了更快地计数，创建一个计数器表，并让您的应用程序根据其执行的插入和删除更新它。然而，在数千个并发事务启动对同一计数器表的更新的情况下，这种方法可能不会很好地扩展。如果近似行数足够，使用 `SHOW TABLE STATUS`。

    `InnoDB` 处理 `SELECT COUNT(*)` 和 `SELECT COUNT(1)` 操作的方式相同。没有性能差异。

    对于 `MyISAM` 表，如果 `SELECT` 从一个表中检索，没有检索到其他列，并且没有 `WHERE` 子句，则 `COUNT(*)` 被优化为非常快速地返回。例如：

    ```sql
    mysql> SELECT COUNT(*) FROM student;
    ```

    这种优化仅适用于 `MyISAM` 表，因为对于这种存储引擎存储了精确的行数计数，并且可以非常快速地访问。如果第一列被定义为 `NOT NULL`，`COUNT(1)` 也仅受到相同优化的影响。

+   [`COUNT(DISTINCT *`expr`*,[*`expr`*...])`](aggregate-functions.html#function_count)

    返回具有不同非`NULL` *`expr`* 值的行数计数。

    如果没有匹配的行，`COUNT(DISTINCT)` 返回 `0`。

    ```sql
    mysql> SELECT COUNT(DISTINCT results) FROM student;
    ```

    在 MySQL 中，您可以通过给出表达式列表来获取不包含 `NULL` 的不同表达式组合的数量。在标准 SQL 中，您将不得不对 `COUNT(DISTINCT ...)` 中的所有表达式进行串联。

+   `GROUP_CONCAT(*`expr`*)`

    此函数返回一个字符串结果，其中包含来自一组的连接的非`NULL` 值。如果没有非`NULL` 值，则返回 `NULL`。完整语法如下：

    ```sql
    GROUP_CONCAT([DISTINCT] *expr* [,*expr* ...]
                 [ORDER BY {*unsigned_integer* | *col_name* | *expr*}
                     [ASC | DESC] [,*col_name* ...]]
                 [SEPARATOR *str_val*])
    ```

    ```sql
    mysql> SELECT student_name,
             GROUP_CONCAT(test_score)
           FROM student
           GROUP BY student_name;
    ```

    或：

    ```sql
    mysql> SELECT student_name,
             GROUP_CONCAT(DISTINCT test_score
                          ORDER BY test_score DESC SEPARATOR ' ')
           FROM student
           GROUP BY student_name;
    ```

    在 MySQL 中，您可以获取表达式组合的连接值。要消除重复值，请使用 `DISTINCT` 子句。要对结果中的值进行排序，请使用 `ORDER BY` 子句。要以相反顺序排序，请在 `ORDER BY` 子句中按照您要排序的列的名称后添加 `DESC`（降序）关键字。默认是升序；这可以通过使用 `ASC` 关键字明确指定。在组中值之间的默认分隔符是逗号（`,`）。要明确指定分隔符，请使用 `SEPARATOR`，后跟应在组值之间插入的字符串文字值。要完全消除分隔符，请指定 `SEPARATOR ''`。

    结果被截断为由`group_concat_max_len`系统变量给出的最大长度，其默认值为 1024。该值可以设置更高，尽管返回值的有效最大长度受`max_allowed_packet`的值限制。在运行时更改`group_concat_max_len`值的语法如下，其中*`val`*是无符号整数：

    ```sql
    SET [GLOBAL | SESSION] group_concat_max_len = *val*;
    ```

    返回值是一个非二进制或二进制字符串，取决于参数是非二进制还是二进制字符串。结果类型是`TEXT`或`BLOB`，除非`group_concat_max_len`小于或等于 512，此时结果类型为`VARCHAR`或`VARBINARY`。

    如果从**mysql**客户端内调用`GROUP_CONCAT()`，二进制字符串结果将使用十六进制表示，取决于`--binary-as-hex`的值。有关该选项的更多信息，请参见第 6.5.1 节，“mysql — MySQL 命令行客户端”。

    另请参阅`CONCAT()`和`CONCAT_WS()`：第 14.8 节，“字符串函数和运算符”。

+   [`JSON_ARRAYAGG(*`col_or_expr`*) [*`over_clause`*]`](aggregate-functions.html#function_json-arrayagg)

    将结果集聚合为一个单一的`JSON`数组，其元素由行组成。此数组中元素的顺序是未定义的。该函数作用于一个列或评估为单个值的表达式。如果结果不包含行或出现错误，则返回`NULL`。如果*`col_or_expr`*为`NULL`，则函数返回一个 JSON 数组，其中包含`[null]`元素。

    从 MySQL 8.0.14 开始，如果存在*`over_clause`*，此函数将作为窗口函数执行。*`over_clause`*的描述如第 14.20.2 节，“窗口函数概念和语法”中所述。

    ```sql
    mysql> SELECT o_id, attribute, value FROM t3;
    +------+-----------+-------+
    | o_id | attribute | value |
    +------+-----------+-------+
    |    2 | color     | red   |
    |    2 | fabric    | silk  |
    |    3 | color     | green |
    |    3 | shape     | square|
    +------+-----------+-------+
    4 rows in set (0.00 sec)

    mysql> SELECT o_id, JSON_ARRAYAGG(attribute) AS attributes
     -> FROM t3 GROUP BY o_id;
    +------+---------------------+
    | o_id | attributes          |
    +------+---------------------+
    |    2 | ["color", "fabric"] |
    |    3 | ["color", "shape"]  |
    +------+---------------------+
    2 rows in set (0.00 sec)
    ```

+   [`JSON_OBJECTAGG(*`key`*, *`value`*) [*`over_clause`*]`](aggregate-functions.html#function_json-objectagg)

    接受两个列名或表达式作为参数，第一个用作键，第二个用作值，并返回一个包含键值对的 JSON 对象。如果结果不包含行，则返回`NULL`，或者在出现错误时返回。如果任何键名为`NULL`或参数数量不等于 2，则会发生错误。

    从 MySQL 8.0.14 开始，如果存在*`over_clause`*，此函数将作为窗口函数执行。*`over_clause`*如第 14.20.2 节，“窗口函数概念和语法”中所述。

    ```sql
    mysql> SELECT o_id, attribute, value FROM t3;
    +------+-----------+-------+
    | o_id | attribute | value |
    +------+-----------+-------+
    |    2 | color     | red   |
    |    2 | fabric    | silk  |
    |    3 | color     | green |
    |    3 | shape     | square|
    +------+-----------+-------+
    4 rows in set (0.00 sec)

    mysql> SELECT o_id, JSON_OBJECTAGG(attribute, value)
     -> FROM t3 GROUP BY o_id;
    +------+---------------------------------------+
    | o_id | JSON_OBJECTAGG(attribute, value)      |
    +------+---------------------------------------+
    |    2 | {"color": "red", "fabric": "silk"}    |
    |    3 | {"color": "green", "shape": "square"} |
    +------+---------------------------------------+
    2 rows in set (0.00 sec)
    ```

    **重复键处理。** 当此函数的结果被规范化时，具有重复键的值将被丢弃。遵循 MySQL `JSON` 数据类型规范，不允许重复键，只使用遇到的最后一个值与该键在返回对象中（“最后重复键获胜”）。这意味着在从`SELECT`中的列使用此函数的结果可能取决于返回行的顺序，这是不被保证的。

    当作为窗口函数使用时，如果帧内存在重复键，结果中只有键的最后一个值。如果`ORDER BY`规范保证值具有特定顺序，则帧中最后一行的键的值是确定的。如果没有，则键的结果值是不确定的。

    考虑以下内容：

    ```sql
    mysql> CREATE TABLE t(c VARCHAR(10), i INT);
    Query OK, 0 rows affected (0.33 sec)

    mysql> INSERT INTO t VALUES ('key', 3), ('key', 4), ('key', 5);
    Query OK, 3 rows affected (0.10 sec)
    Records: 3  Duplicates: 0  Warnings: 0

    mysql> SELECT c, i FROM t;
    +------+------+
    | c    | i    |
    +------+------+
    | key  |    3 |
    | key  |    4 |
    | key  |    5 |
    +------+------+
    3 rows in set (0.00 sec)

    mysql> SELECT JSON_OBJECTAGG(c, i) FROM t;
    +----------------------+
    | JSON_OBJECTAGG(c, i) |
    +----------------------+
    | {"key": 5}           |
    +----------------------+
    1 row in set (0.00 sec)

    mysql> DELETE FROM t;
    Query OK, 3 rows affected (0.08 sec)

    mysql> INSERT INTO t VALUES ('key', 3), ('key', 5), ('key', 4);
    Query OK, 3 rows affected (0.06 sec)
    Records: 3  Duplicates: 0  Warnings: 0

    mysql> SELECT c, i FROM t;
    +------+------+
    | c    | i    |
    +------+------+
    | key  |    3 |
    | key  |    5 |
    | key  |    4 |
    +------+------+
    3 rows in set (0.00 sec)

    mysql> SELECT JSON_OBJECTAGG(c, i) FROM t;
    +----------------------+
    | JSON_OBJECTAGG(c, i) |
    +----------------------+
    | {"key": 4}           |
    +----------------------+
    1 row in set (0.00 sec)
    ```

    从上次查询中选择的键是不确定的。如果查询不使用`GROUP BY`（通常会强加自己的排序），并且您希望特定键的顺序，您可以通过在`OVER`子句中包含一个`ORDER BY`规范来将`JSON_OBJECTAGG()`作为窗口函数调用，以对帧行施加特定顺序。以下示例展示了对于几种不同帧规范，使用和不使用`ORDER BY`会发生什么。

    没有`ORDER BY`，帧是整个分区：

    ```sql
    mysql> SELECT JSON_OBJECTAGG(c, i)
           OVER () AS json_object FROM t;
    +-------------+
    | json_object |
    +-------------+
    | {"key": 4}  |
    | {"key": 4}  |
    | {"key": 4}  |
    +-------------+
    ```

    使用`ORDER BY`，其中帧是默认的`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`（无论升序还是降序）：

    ```sql
    mysql> SELECT JSON_OBJECTAGG(c, i)
           OVER (ORDER BY i) AS json_object FROM t;
    +-------------+
    | json_object |
    +-------------+
    | {"key": 3}  |
    | {"key": 4}  |
    | {"key": 5}  |
    +-------------+
    mysql> SELECT JSON_OBJECTAGG(c, i)
           OVER (ORDER BY i DESC) AS json_object FROM t;
    +-------------+
    | json_object |
    +-------------+
    | {"key": 5}  |
    | {"key": 4}  |
    | {"key": 3}  |
    +-------------+
    ```

    使用`ORDER BY`和整个分区的显式帧：

    ```sql
    mysql> SELECT JSON_OBJECTAGG(c, i)
           OVER (ORDER BY i
                ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
            AS json_object
           FROM t;
    +-------------+
    | json_object |
    +-------------+
    | {"key": 5}  |
    | {"key": 5}  |
    | {"key": 5}  |
    +-------------+
    ```

    要返回特定键值（例如最小值或最大值），请在适当查询中包含一个`LIMIT`子句。例如：

    ```sql
    mysql> SELECT JSON_OBJECTAGG(c, i)
           OVER (ORDER BY i) AS json_object FROM t LIMIT 1;
    +-------------+
    | json_object |
    +-------------+
    | {"key": 3}  |
    +-------------+
    mysql> SELECT JSON_OBJECTAGG(c, i)
           OVER (ORDER BY i DESC) AS json_object FROM t LIMIT 1;
    +-------------+
    | json_object |
    +-------------+
    | {"key": 5}  |
    +-------------+
    ```

    有关 JSON 值的规范化、合并和自动包装，请参阅 JSON 值的规范化、合并和自动包装，获取更多信息和示例。

+   [`MAX([DISTINCT] *`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_max)

    返回*`expr`*的最大值。`MAX()`可能接受一个字符串参数；在这种情况下，它返回最大的字符串值。参见第 10.3.1 节，“MySQL 如何使用索引”。`DISTINCT`关键字可用于查找*`expr`*的不同值的最大值，但是，这与省略`DISTINCT`产生相同的结果。

    如果没有匹配的行，或者*`expr`*为`NULL`，`MAX()`返回`NULL`。

    如果*`over_clause`*存在，则此函数作为窗口函数执行。*`over_clause`*如第 14.20.2 节，“窗口函数概念和语法”中所述；它不能与`DISTINCT`一起使用。

    ```sql
    mysql> SELECT student_name, MIN(test_score), MAX(test_score)
           FROM student
           GROUP BY student_name;
    ```

    对于`MAX()`，MySQL 当前通过它们的字符串值而不是字符串在集合中的相对位置来比较`ENUM`和`SET`列。这与`ORDER BY`比较它们的方式不同。

+   [`MIN([DISTINCT] *`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_min)

    返回*`expr`*的最小值。`MIN()`可能接受一个字符串参数；在这种情况下，它返回最小的字符串值。参见第 10.3.1 节，“MySQL 如何使用索引”。`DISTINCT`关键字可用于查找*`expr`*的不同值的最小值，但是，这与省略`DISTINCT`产生相同的结果。

    如果没有匹配的行，或者*`expr`*为`NULL`，`MIN()`返回`NULL`。

    如果*`over_clause`*存在，则此函数作为窗口函数执行。*`over_clause`*如第 14.20.2 节，“窗口函数概念和语法”中所述；它不能与`DISTINCT`一起使用。

    ```sql
    mysql> SELECT student_name, MIN(test_score), MAX(test_score)
           FROM student
           GROUP BY student_name;
    ```

    对于`MIN()`，MySQL 当前通过它们的字符串值而不是字符串在集合中的相对位置来比较`ENUM`和`SET`列。这与`ORDER BY`比较它们的方式不同。

+   [`STD(*`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_std)

    返回*`expr`*的总体标准偏差。`STD()`是标准 SQL 函数`STDDEV_POP()`的同义词，作为 MySQL 的扩展提供。

    如果没有匹配的行，或者*`expr`*为`NULL`，`STD()`返回`NULL`。

    如果存在*`over_clause`*，此函数将作为窗口函数执行。*`over_clause`*的描述如第 14.20.2 节，“窗口函数概念和语法”中所述。

+   [`STDDEV(*`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_stddev)

    返回*`expr`*的总体标准偏差。`STDDEV()`是标准 SQL 函数`STDDEV_POP()`的同义词，用于与 Oracle 兼容。

    如果没有匹配的行，或者*`expr`*为`NULL`，`STDDEV()`返回`NULL`。

    如果存在*`over_clause`*，此函数将作为窗口函数执行。*`over_clause`*的描述如第 14.20.2 节，“窗口函数概念和语法”中所述。

+   [`STDDEV_POP(*`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_stddev-pop)

    返回*`expr`*的总体标准偏差（`VAR_POP()`的平方根）。您还可以使用`STD()`或`STDDEV()`，它们是等效的但不是标准 SQL。

    如果没有匹配的行，或者*`expr`*为`NULL`，`STDDEV_POP()`返回`NULL`。

    如果存在*`over_clause`*，此函数将作为窗口函数执行。*`over_clause`*的描述如第 14.20.2 节，“窗口函数概念和语法”中所述。

+   [`STDDEV_SAMP(*`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_stddev-samp)

    返回*`expr`*的样本标准偏差（`VAR_SAMP()`的平方根）。

    如果没有匹配的行，或者*`expr`*为`NULL`，`STDDEV_SAMP()`返回`NULL`。

    如果存在*`over_clause`*，此函数将作为窗口函数执行。*`over_clause`*的描述如第 14.20.2 节，“窗口函数概念和语法”中所述。

+   [`SUM([DISTINCT] *`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_sum)

    返回*`expr`*的总和。如果返回集没有行，则`SUM()`返回`NULL`。可以使用`DISTINCT`关键字仅对*`expr`*的不同值求和。

    如果没有匹配的行，或者*`expr`*为`NULL`，`SUM()`返回`NULL`。

    如果存在*`over_clause`*，此函数将作为窗口函数执行。*`over_clause`*如第 14.20.2 节，“窗口函数概念和语法”中描述的那样；它不能与`DISTINCT`一起使用。

+   [`VAR_POP(*`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_var-pop)

    返回*`expr`*的总体标准方差。它将行视为整体总体，而不是样本，因此分母是行数。您也可以使用`VARIANCE()`，它是等效的但不是标准 SQL。

    如果没有匹配的行，或者*`expr`*为`NULL`，`VAR_POP()`返回`NULL`。

    如果存在*`over_clause`*，此函数将作为窗口函数执行。*`over_clause`*如第 14.20.2 节，“窗口函数概念和语法”中描述的那样。

+   [`VAR_SAMP(*`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_var-samp)

    返回*`expr`*的样本方差。也就是说，分母是行数减一。

    如果没有匹配的行，或者*`expr`*为`NULL`，`VAR_SAMP()`返回`NULL`。

    如果存在*`over_clause`*，此函数将作为窗口函数执行。*`over_clause`*如第 14.20.2 节，“窗口函数概念和语法”中描述的那样。

+   [`VARIANCE(*`expr`*) [*`over_clause`*]`](aggregate-functions.html#function_variance)

    返回*`expr`*的总体标准方差。`VARIANCE()`是标准 SQL 函数`VAR_POP()`的同义词，作为 MySQL 扩展提供。

    如果没有匹配的行，或者*`expr`*为`NULL`，`VARIANCE()`返回`NULL`。

    如果存在*`over_clause`*，此函数将作为窗口函数执行。*`over_clause`*如第 14.20.2 节，“窗口函数概念和语法”中描述的那样。
