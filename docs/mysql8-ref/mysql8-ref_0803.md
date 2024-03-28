# 14.8.1 字符串比较函数和运算符

> 原文：[`dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html)

**表 14.13 字符串比较函数和运算符**

| 名称 | 描述 |
| --- | --- |
| `LIKE` | 简单模式匹配 |
| `NOT LIKE` | 简单模式匹配的否定 |
| `STRCMP()` | 比较两个字符串 |

如果将二进制字符串作为参数传递给字符串函数，则结果字符串也是二进制字符串。将数字转换为字符串时被视为二进制字符串。这仅影响比较。

通常，如果字符串比较中的任何表达式是区分大小写的，则比较将以区分大小写的方式执行。

如果在 **mysql** 客户端中调用字符串函数，则二进制字符串将以十六进制表示形式显示，具体取决于 `--binary-as-hex` 的值。有关该选项的更多信息，请参见 第 6.5.1 节，“mysql — MySQL 命令行客户端”。

+   [`*`expr`* LIKE *`pat`* [ESCAPE '*`escape_char`*']`](string-comparison-functions.html#operator_like)

    使用 SQL 模式进行模式匹配。返回 `1`（`TRUE`）或 `0`（`FALSE`）。如果 *`expr`* 或 *`pat`* 中的任一者是 `NULL`，结果为 `NULL`。

    模式不一定是一个字面字符串。例如，它可以被指定为一个字符串表达式或表列。在后一种情况下，该列必须被定义为 MySQL 字符串类型之一（参见 第 13.3 节，“字符串数据类型”）。

    根据 SQL 标准，`LIKE` 按字符进行匹配，因此它可能产生与 `=` 比较运算符不同的结果：

    ```sql
    mysql> SELECT 'ä' LIKE 'ae' COLLATE latin1_german2_ci;
    +-----------------------------------------+
    | 'ä' LIKE 'ae' COLLATE latin1_german2_ci |
    +-----------------------------------------+
    |                                       0 |
    +-----------------------------------------+
    mysql> SELECT 'ä' = 'ae' COLLATE latin1_german2_ci;
    +--------------------------------------+
    | 'ä' = 'ae' COLLATE latin1_german2_ci |
    +--------------------------------------+
    |                                    1 |
    +--------------------------------------+
    ```

    特别是，尾随空格始终是有意义的。这与使用 `=` 运算符执行的比较不同，对于非二进制字符串（`CHAR`、`VARCHAR` 和 `TEXT` 值）的尾随空格的重要性取决于用于比较的排序规则的填充属性。有关更多信息，请参见 比较中的尾随空格处理。

    使用 `LIKE` 可以在模式中使用以下两个通配符：

    +   `%` 匹配任意数量的字符，甚至零个字符。

    +   `_` 匹配正好一个字符。

    ```sql
    mysql> SELECT 'David!' LIKE 'David_';
     -> 1
    mysql> SELECT 'David!' LIKE '%D%v%';
     -> 1
    ```

    要测试通配符字符的文字实例，需在其前面加上转义字符。如果不指定`ESCAPE`字符，则假定为`\`，除非启用了`NO_BACKSLASH_ESCAPES` SQL 模式。在这种情况下，不使用转义字符。

    +   `\%`匹配一个`%`字符。

    +   `\_`匹配一个`_`字符。

    ```sql
    mysql> SELECT 'David!' LIKE 'David\_';
     -> 0
    mysql> SELECT 'David_' LIKE 'David\_';
     -> 1
    ```

    要指定不同的转义字符，请使用`ESCAPE`子句：

    ```sql
    mysql> SELECT 'David_' LIKE 'David|_' ESCAPE '|';
     -> 1
    ```

    转义序列应为一个字符长，用于指定转义字符，或为空以指定不使用转义字符。表达式必须在执行时评估为常量。如果启用了`NO_BACKSLASH_ESCAPES` SQL 模式，则序列不能是空的。

    以下语句说明字符串比较不区分大小写，除非操作数之一是区分大小写的（使用区分大小写的排序规则或是二进制字符串）：

    ```sql
    mysql> SELECT 'abc' LIKE 'ABC';
     -> 1
    mysql> SELECT 'abc' LIKE _utf8mb4 'ABC' COLLATE utf8mb4_0900_as_cs;
     -> 0
    mysql> SELECT 'abc' LIKE _utf8mb4 'ABC' COLLATE utf8mb4_bin;
     -> 0
    mysql> SELECT 'abc' LIKE BINARY 'ABC';
     -> 0
    ```

    作为对标准 SQL 的扩展，MySQL 允许在数值表达式上使用`LIKE`。

    ```sql
    mysql> SELECT 10 LIKE '1%';
     -> 1
    ```

    在这种情况下，MySQL 尝试对表达式进行隐式转换为字符串。参见第 14.3 节，“表达式评估中的类型转换”。

    注意

    MySQL 在字符串中使用 C 转义语法（例如，`\n`表示换行符）。如果要让`LIKE`字符串包含文字`\`，必须将其双写。（除非启用了`NO_BACKSLASH_ESCAPES` SQL 模式，在这种情况下不使用转义字符。）例如，要搜索`\n`，请指定为`\\n`。要搜索`\`，请指定为`\\\\`；这是因为解析器会将反斜杠剥离一次，然后在进行模式匹配时再次剥离，留下一个单独的反斜杠进行匹配。

    例外情况：在模式字符串的末尾，可以指定反斜杠为`\\`。在字符串的末尾，反斜杠代表自身，因为后面没有需要转义的内容。假设一个表包含以下值：

    ```sql
    mysql> SELECT filename FROM t1;
    +--------------+
    | filename     |
    +--------------+
    | C:           |
    | C:\          |
    | C:\Programs  |
    | C:\Programs\ |
    +--------------+
    ```

    要测试以反斜杠结尾的值，可以使用以下任一模式匹配这些值：

    ```sql
    mysql> SELECT filename, filename LIKE '%\\' FROM t1;
    +--------------+---------------------+
    | filename     | filename LIKE '%\\' |
    +--------------+---------------------+
    | C:           |                   0 |
    | C:\          |                   1 |
    | C:\Programs  |                   0 |
    | C:\Programs\ |                   1 |
    +--------------+---------------------+

    mysql> SELECT filename, filename LIKE '%\\\\' FROM t1;
    +--------------+-----------------------+
    | filename     | filename LIKE '%\\\\' |
    +--------------+-----------------------+
    | C:           |                     0 |
    | C:\          |                     1 |
    | C:\Programs  |                     0 |
    | C:\Programs\ |                     1 |
    +--------------+-----------------------+
    ```

+   [`*`expr`* NOT LIKE *`pat`* [ESCAPE '*`escape_char`*']`](string-comparison-functions.html#operator_not-like)

    这与`NOT (*`expr`* LIKE *`pat`* [ESCAPE '*`escape_char`*'])`相同。

    注意

    包含`NULL`的列进行`NOT LIKE`比较的聚合查询可能产生意外结果。例如，考虑以下表和数据：

    ```sql
    CREATE TABLE foo (bar VARCHAR(10));

    INSERT INTO foo VALUES (NULL), (NULL);
    ```

    查询 `SELECT COUNT(*) FROM foo WHERE bar LIKE '%baz%';` 返回 `0`。你可能会认为 `SELECT COUNT(*) FROM foo WHERE bar NOT LIKE '%baz%';` 会返回 `2`。然而，事实并非如此：第二个查询返回 `0`。这是因为 `NULL NOT LIKE *`expr`*` 总是返回 `NULL`，无论 *`expr`* 的值如何。对于涉及 `NULL` 和使用 `NOT RLIKE` 或 `NOT REGEXP` 进行比较的聚合查询，必须显式测试 `NOT NULL`，使用 `OR`（而不是 `AND`），如下所示：

    ```sql
    SELECT COUNT(*) FROM foo WHERE bar NOT LIKE '%baz%' OR bar IS NULL;
    ```

+   `STRCMP(*`expr1`*,*`expr2`*)`

    `STRCMP()` 如果字符串相同则返回 `0`，如果第一个参数根据当前排序顺序小于第二个参数则返回 `-1`，如果任一参数为 `NULL` 则返回 `NULL`，否则返回 `1`。

    ```sql
    mysql> SELECT STRCMP('text', 'text2');
     -> -1
    mysql> SELECT STRCMP('text2', 'text');
     -> 1
    mysql> SELECT STRCMP('text', 'text');
     -> 0
    ```

    `STRCMP()` 使用参数的排序规则执行比较。

    ```sql
    mysql> SET @s1 = _utf8mb4 'x' COLLATE utf8mb4_0900_ai_ci;
    mysql> SET @s2 = _utf8mb4 'X' COLLATE utf8mb4_0900_ai_ci;
    mysql> SET @s3 = _utf8mb4 'x' COLLATE utf8mb4_0900_as_cs;
    mysql> SET @s4 = _utf8mb4 'X' COLLATE utf8mb4_0900_as_cs;
    mysql> SELECT STRCMP(@s1, @s2), STRCMP(@s3, @s4);
    +------------------+------------------+
    | STRCMP(@s1, @s2) | STRCMP(@s3, @s4) |
    +------------------+------------------+
    |                0 |               -1 |
    +------------------+------------------+
    ```

    如果排序规则不兼容，则其中一个参数必须转换为与另一个兼容。参见 Section 12.8.4, “Collation Coercibility in Expressions”。

    ```sql
    mysql> SET @s1 = _utf8mb4 'x' COLLATE utf8mb4_0900_ai_ci;
    mysql> SET @s2 = _utf8mb4 'X' COLLATE utf8mb4_0900_ai_ci;
    mysql> SET @s3 = _utf8mb4 'x' COLLATE utf8mb4_0900_as_cs;
    mysql> SET @s4 = _utf8mb4 'X' COLLATE utf8mb4_0900_as_cs;
    -->
    mysql> SELECT STRCMP(@s1, @s3);
    ERROR 1267 (HY000): Illegal mix of collations (utf8mb4_0900_ai_ci,IMPLICIT)
    and (utf8mb4_0900_as_cs,IMPLICIT) for operation 'strcmp'
    mysql> SELECT STRCMP(@s1, @s3 COLLATE utf8mb4_0900_ai_ci);
    +---------------------------------------------+
    | STRCMP(@s1, @s3 COLLATE utf8mb4_0900_ai_ci) |
    +---------------------------------------------+
    |                                           0 |
    +---------------------------------------------+
    ```
