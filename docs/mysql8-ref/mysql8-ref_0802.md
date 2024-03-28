# 14.8 字符串函数和运算符

> 原文：[`dev.mysql.com/doc/refman/8.0/en/string-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html)

14.8.1 字符串比较函数和运算符

14.8.2 正则表达式

14.8.3 函数结果的字符集和排序规则

**表 14.12 字符串函数和运算符**

| 名称 | 描述 |
| --- | --- |
| `ASCII()` | 返回最左字符的数值 |
| `BIN()` | 返回包含数字二进制表示的字符串 |
| `BIT_LENGTH()` | 返回参数的位长度 |
| `CHAR()` | 返回每个传递整数的字符 |
| `CHAR_LENGTH()` | 返回参数中的字符数 |
| `CHARACTER_LENGTH()` | CHAR_LENGTH()的同义词 |
| `CONCAT()` | 返回连接的字符串 |
| `CONCAT_WS()` | 返回带有分隔符的连接字符串 |
| `ELT()` | 返回索引号处的字符串 |
| `EXPORT_SET()` | 返回一个字符串，对于值位设置的每个位，您会得到一个打开的字符串，对于未设置的位，您会得到一个关闭的字符串 |
| `FIELD()` | 第一个参数在后续参数中的索引（位置） |
| `FIND_IN_SET()` | 第一个参数在第二个参数中的索引（位置） |
| `FORMAT()` | 返回格式化为指定小数位数的数字 |
| `FROM_BASE64()` | 解码 base64 编码的字符串并返回结果 |
| `HEX()` | 十进制或字符串值的十六进制表示 |
| `INSERT()` | 在指定位置插入子字符串，最多指定数量的字符 |
| `INSTR()` | 返回子字符串的第一个出现的索引 |
| `LCASE()` | LOWER()的同义词 |
| `LEFT()` | 返回指定数量的最左字符 |
| `LENGTH()` | 返回字符串的字节长度 |
| `LIKE` | 简单的模式匹配 |
| `LOAD_FILE()` | 加载指定文件 |
| `LOCATE()` | 返回子字符串的第一个出现位置 |
| `LOWER()` | 返回小写参数 |
| `LPAD()` | 返回左侧填充了指定字符串的字符串参数 |
| `LTRIM()` | 移除前导空格 |
| `MAKE_SET()` | 返回一组逗号分隔的字符串，其中对应位在 bits 中设置 |
| `MATCH()` | 执行全文搜索 |
| `MID()` | 返回从指定位置开始的子字符串 |
| `NOT LIKE` | 简单模式匹配的否定 |
| `NOT REGEXP` | REGEXP 的否定 |
| `OCT()` | 返回包含数字的八进制表示的字符串 |
| `OCTET_LENGTH()` | LENGTH() 的同义词 |
| `ORD()` | 返回参数的最左字符的字符代码 |
| `POSITION()` | LOCATE() 的同义词 |
| `QUOTE()` | 为在 SQL 语句中使用而转义参数 |
| `REGEXP` | 字符串是否匹配正则表达式 |
| `REGEXP_INSTR()` | 返回匹配正则表达式的子字符串的起始索引 |
| `REGEXP_LIKE()` | 字符串是否匹配正则表达式 |
| `REGEXP_REPLACE()` | 替换与正则表达式匹配的子字符串 |
| `REGEXP_SUBSTR()` | 返回匹配正则表达式的子字符串 |
| `REPEAT()` | 重复指定次数的字符串 |
| `REPLACE()` | 替换指定字符串的出现次数 |
| `REVERSE()` | 反转字符串中的字符 |
| `RIGHT()` | 返回指定右侧字符数 |
| `RLIKE` | 字符串是否匹配正则表达式 |
| `RPAD()` | 追加指定次数的字符串 |
| `RTRIM()` | 移除尾部空格 |
| `SOUNDEX()` | 返回一个 Soundex 字符串 |
| `SOUNDS LIKE` | 比较声音 |
| `SPACE()` | 返回指定数量的空格字符串 |
| `STRCMP()` | 比较两个字符串 |
| `SUBSTR()` | 返回指定的子字符串 |
| `SUBSTRING()` | 返回指定的子字符串 |
| `SUBSTRING_INDEX()` | 返回指定分隔符之前的字符串子串 |
| `TO_BASE64()` | 返回转换为 base-64 字符串的参数 |
| `TRIM()` | 删除前导和尾随空格 |
| `UCASE()` | UPPER()的同义词 |
| `UNHEX()` | 返回包含数字的十六进制表示的字符串 |
| `UPPER()` | 转换为大写 |
| `WEIGHT_STRING()` | 返回字符串的权重字符串 |
| 名称 | 描述 |

如果结果的长度大于`max_allowed_packet`系统变量的值，则返回`NULL`。有关详细信息，请参阅第 7.1.1 节，“配置服务器”。

对于操作字符串位置的函数，第一个位置编号为 1。

对于需要长度参数的函数，非整数参数将四舍五入为最接近的整数。

+   `ASCII(*`str`*)`

    返回字符串*`str`*的最左字符的数值。如果*`str`*为空字符串，则返回`0`。如果*`str`*为`NULL`，则返回`NULL`。`ASCII()`适用于 8 位字符。

    ```sql
    mysql> SELECT ASCII('2');
     -> 50
    mysql> SELECT ASCII(2);
     -> 50
    mysql> SELECT ASCII('dx');
     -> 100
    ```

    另请参见`ORD()`函数。

+   `BIN(*`N`*)`

    返回*`N`*的二进制值的字符串表示，其中*`N`*是长整型(`BIGINT`)数字。这等效于`CONV(*`N`*,10,2)`。如果*`N`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT BIN(12);
     -> '1100'
    ```

+   `BIT_LENGTH(*`str`*)`

    返回字符串*`str`*的位数。如果*`str`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT BIT_LENGTH('text');
     -> 32
    ```

+   [`CHAR(*`N`*,... [USING *`charset_name`*])`](string-functions.html#function_char)

    `CHAR()`将每个参数*`N`*解释为整数，并返回由这些整数的代码值给出的字符组成的字符串。跳过`NULL`值。

    ```sql
    mysql> SELECT CHAR(77,121,83,81,'76');
    +--------------------------------------------------+
    | CHAR(77,121,83,81,'76')                          |
    +--------------------------------------------------+
    | 0x4D7953514C                                     |
    +--------------------------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT CHAR(77,77.3,'77.3');
    +--------------------------------------------+
    | CHAR(77,77.3,'77.3')                       |
    +--------------------------------------------+
    | 0x4D4D4D                                   |
    +--------------------------------------------+
    1 row in set (0.00 sec)
    ```

    默认情况下，`CHAR()`返回一个二进制字符串。要生成给定字符集中的字符串，请使用可选的`USING`子句：

    ```sql
    mysql> SELECT CHAR(77,121,83,81,'76' USING utf8mb4);
    +---------------------------------------+
    | CHAR(77,121,83,81,'76' USING utf8mb4) |
    +---------------------------------------+
    | MySQL                                 |
    +---------------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT CHAR(77,77.3,'77.3' USING utf8mb4);
    +------------------------------------+
    | CHAR(77,77.3,'77.3' USING utf8mb4) |
    +------------------------------------+
    | MMM                                |
    +------------------------------------+
    1 row in set, 1 warning (0.00 sec)

    mysql> SHOW WARNINGS;
    +---------+------+-------------------------------------------+
    | Level   | Code | Message                                   |
    +---------+------+-------------------------------------------+
    | Warning | 1292 | Truncated incorrect INTEGER value: '77.3' |
    +---------+------+-------------------------------------------+
    1 row in set (0.00 sec)
    ```

    如果提供了`USING`并且结果字符串对于给定的字符集是非法的，则会发出警告。此外，如果启用了严格的 SQL 模式，则`CHAR()`的结果将变为`NULL`。

    如果从**mysql**客户端内调用`CHAR()`，二进制字符串将根据`--binary-as-hex`的值以十六进制表示。有关该选项的更多信息，请参见 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

    大于 255 的`CHAR()`参数将转换为多个结果字节。例如，`CHAR(256)`等同于`CHAR(1,0)`，而`CHAR(256*256)`等同于`CHAR(1,0,0)`：

    ```sql
    mysql> SELECT HEX(CHAR(1,0)), HEX(CHAR(256));
    +----------------+----------------+
    | HEX(CHAR(1,0)) | HEX(CHAR(256)) |
    +----------------+----------------+
    | 0100           | 0100           |
    +----------------+----------------+
    1 row in set (0.00 sec)

    mysql> SELECT HEX(CHAR(1,0,0)), HEX(CHAR(256*256));
    +------------------+--------------------+
    | HEX(CHAR(1,0,0)) | HEX(CHAR(256*256)) |
    +------------------+--------------------+
    | 010000           | 010000             |
    +------------------+--------------------+
    1 row in set (0.00 sec)
    ```

+   `CHAR_LENGTH(*`str`*)`

    返回*`str`*的字符串长度，以代码点计算。多字节字符计为一个代码点。这意味着，对于包含两个 3 字节字符的字符串，`LENGTH()`返回`6`，而`CHAR_LENGTH()`返回`2`，如下所示：

    ```sql
    mysql> SET @dolphin:='海豚';
    Query OK, 0 rows affected (0.01 sec)

    mysql> SELECT LENGTH(@dolphin), CHAR_LENGTH(@dolphin);
    +------------------+-----------------------+
    | LENGTH(@dolphin) | CHAR_LENGTH(@dolphin) |
    +------------------+-----------------------+
    |                6 |                     2 |
    +------------------+-----------------------+
    1 row in set (0.00 sec)
    ```

    如果*`str`*为`NULL`，则`CHAR_LENGTH()`返回`NULL`。

+   `CHARACTER_LENGTH(*`str`*)`

    `CHARACTER_LENGTH()`是`CHAR_LENGTH()`的同义词。

+   `CONCAT(*`str1`*,*`str2`*,...)`

    返回从连接参数结果的字符串。可能有一个或多个参数。如果所有参数都是非二进制字符串，则结果是非二进制字符串。如果参数包含任何二进制字符串，则结果是二进制字符串。数值参数转换为其等效的非二进制字符串形式。

    如果任何参数为`NULL`，`CONCAT()`返回`NULL`。

    ```sql
    mysql> SELECT CONCAT('My', 'S', 'QL');
     -> 'MySQL'
    mysql> SELECT CONCAT('My', NULL, 'QL');
     -> NULL
    mysql> SELECT CONCAT(14.3);
     -> '14.3'
    ```

    对于带引号的字符串，可以通过将字符串放在一起来执行连接：

    ```sql
    mysql> SELECT 'My' 'S' 'QL';
     -> 'MySQL'
    ```

    如果从**mysql**客户端内调用`CONCAT()`，二进制字符串结果将根据`--binary-as-hex`的值以十六进制表示。有关该选项的更多信息，请参见 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

+   `CONCAT_WS(*`分隔符`*,*`str1`*,*`str2`*,...)`

    `CONCAT_WS()`代表带分隔符连接，是`CONCAT()`的特殊形式。第一个参数是其余参数的分隔符。分隔符添加在要连接的字符串之间。分隔符可以是字符串，其余参数也可以是字符串。如果分隔符为`NULL`，则结果为`NULL`。

    ```sql
    mysql> SELECT CONCAT_WS(',','First name','Second name','Last Name');
     -> 'First name,Second name,Last Name'
    mysql> SELECT CONCAT_WS(',','First name',NULL,'Last Name');
     -> 'First name,Last Name'
    ```

    `CONCAT_WS()`不会跳过空字符串。但是，在分隔符参数之后，它会跳过任何`NULL`值。

+   `ELT(*`N`*,*`str1`*,*`str2`*,*`str3`*,...)`

    `ELT()`返回字符串列表中第*`N`*个元素：如果*`N`* = `1`，则返回*`str1`*，如果*`N`* = `2`，则返回*`str2`*，依此类推。如果*`N`*小于`1`，大于参数数量或为`NULL`，则返回`NULL`。`ELT()`是`FIELD()`的补充。

    ```sql
    mysql> SELECT ELT(1, 'Aa', 'Bb', 'Cc', 'Dd');
     -> 'Aa'
    mysql> SELECT ELT(4, 'Aa', 'Bb', 'Cc', 'Dd');
     -> 'Dd'
    ```

+   [`EXPORT_SET(*`bits`*,*`on`*,*`off`*[,*`separator`*[,*`number_of_bits`*]])`](string-functions.html#function_export-set)

    返回一个字符串，对于值*`bits`*中设置的每个位，您会得到一个*`on`*字符串，对于值中未设置的每个位，您会得到一个*`off`*字符串。从右到左（从低位到高位）检查*`bits`*中的位。字符串从左到右添加到结果中，由*`separator`*字符串分隔（默认为逗号字符`,`）。检查的位数由*`number_of_bits`*给出，如果未指定，默认为`64`。如果*`number_of_bits`*大于`64`，则会被静默截断为`64`。它被视为无符号整数，因此值为−1 实际上等同于`64`。

    ```sql
    mysql> SELECT EXPORT_SET(5,'Y','N',',',4);
     -> 'Y,N,Y,N'
    mysql> SELECT EXPORT_SET(6,'1','0',',',10);
     -> '0,1,1,0,0,0,0,0,0,0'
    ```

+   `FIELD(*`str`*,*`str1`*,*`str2`*,*`str3`*,...)`

    返回*`str`*在*`str1`*、*`str2`*、*`str3`*、`...`列表中的索引（位置）。如果未找到*`str`*，则返回`0`。

    如果`FIELD()`的所有参数都是字符串，则所有参数将作为字符串进行比较。如果所有参数都是数字，则它们将作为数字进行比较。否则，参数将作为双精度数进行比较。

    如果*`str`*为`NULL`，则返回值为`0`，因为`NULL`与任何值的相等比较失败。`FIELD()`是`ELT()`的补充。

    ```sql
    mysql> SELECT FIELD('Bb', 'Aa', 'Bb', 'Cc', 'Dd', 'Ff');
     -> 2
    mysql> SELECT FIELD('Gg', 'Aa', 'Bb', 'Cc', 'Dd', 'Ff');
     -> 0
    ```

+   `FIND_IN_SET(*`str`*,*`strlist`*)`

    如果字符串*`str`*在由*`N`*个子字符串组成的字符串列表*`strlist`*中，则返回 1 到*`N`*的值。字符串列表是由`,`字符分隔的子字符串组成的字符串。如果第一个参数是常量字符串，第二个参数是`SET`类型的列，则`FIND_IN_SET()`函数会优化使用位运算。如果*`str`*不在*`strlist`*中或*`strlist`*为空字符串，则返回`0`。如果任一参数为`NULL`，则返回`NULL`。如果第一个参数包含逗号（`,`）字符，则此函数无法正常工作。

    ```sql
    mysql> SELECT FIND_IN_SET('b','a,b,c,d');
     -> 2
    ```

+   [`FORMAT(*`X`*,*`D`*[,*`locale`*])`](string-functions.html#function_format)

    将数字*`X`*格式化为类似`'#,###,###.##'`的格式，四舍五入到*`D`*位小数，并将结果作为字符串返回。如果*`D`*为`0`，则结果没有小数点或小数部分。如果*`X`*或*`D`*为`NULL`，则函数返回`NULL`。

    可选的第三个参数允许指定要用于结果数字的小数点、千位分隔符和分隔符之间的分组的区域设置。允许的区域设置值与`lc_time_names`系统变量的合法值相同（请参阅第 12.16 节，“MySQL 服务器区域设置支持”）。如果区域设置为`NULL`或未指定，则默认区域设置为`'en_US'`。

    ```sql
    mysql> SELECT FORMAT(12332.123456, 4);
     -> '12,332.1235'
    mysql> SELECT FORMAT(12332.1,4);
     -> '12,332.1000'
    mysql> SELECT FORMAT(12332.2,0);
     -> '12,332'
    mysql> SELECT FORMAT(12332.2,2,'de_DE');
     -> '12.332,20'
    ```

+   `FROM_BASE64(*`str`*)`

    将一个使用`TO_BASE64()`使用的 base-64 编码规则编码的字符串解码，并将解码结果作为二进制字符串返回。如果参数为`NULL`或不是有效的 base-64 字符串，则结果为`NULL`。有关编码和解码规则的详细信息，请参阅`TO_BASE64()`的描述。

    ```sql
    mysql> SELECT TO_BASE64('abc'), FROM_BASE64(TO_BASE64('abc'));
     -> 'JWJj', 'abc'
    ```

    如果在**mysql**客户端内部调用`FROM_BASE64()`，二进制字符串将使用十六进制表示。您可以通过在启动**mysql**客户端时将`--binary-as-hex`的值设置为`0`来禁用此行为。有关该选项的更多信息，请参阅第 6.5.1 节，“mysql — The MySQL Command-Line Client”。

+   `HEX(*`str`*)`, `HEX(*`N`*)`

    对于字符串参数*`str`*，`HEX()` 返回一个十六进制字符串表示*`str`*，其中*`str`*中每个字符的每个字节都转换为两个十六进制数字。（因此，多字节字符会变成两个以上的数字。）此操作的逆操作由`UNHEX()` 函数执行。

    对于数值参数*`N`*，`HEX()` 返回将*`N`*的值视为长整型（`BIGINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")）数字的十六进制字符串表示。这等效于`CONV(*`N`*,10,16)`。此操作的逆操作由`CONV(HEX(*`N`*),16,10)`执行。

    对于`NULL`参数，此函数返回`NULL`。

    ```sql
    mysql> SELECT X'616263', HEX('abc'), UNHEX(HEX('abc'));
     -> 'abc', 616263, 'abc'
    mysql> SELECT HEX(255), CONV(HEX(255),16,10);
     -> 'FF', 255
    ```

+   `INSERT(*`str`*,*`pos`*,*`len`*,*`newstr`*)`

    返回字符串*`str`*，从位置*`pos`*开始的子字符串，长度为*`len`*个字符，被字符串*`newstr`*替换。如果*`pos`*不在字符串长度范围内，则返回原始字符串。如果*`len`*不在剩余字符串的长度范围内，则从位置*`pos`*开始替换剩余字符串。如果任何参数为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT INSERT('Quadratic', 3, 4, 'What');
     -> 'QuWhattic'
    mysql> SELECT INSERT('Quadratic', -1, 4, 'What');
     -> 'Quadratic'
    mysql> SELECT INSERT('Quadratic', 3, 100, 'What');
     -> 'QuWhat'
    ```

    此函数支持多字节字符。

+   `INSTR(*`str`*,*`substr`*)`

    返回字符串*`str`*中子字符串*`substr`*第一次出现的位置。这与`LOCATE()`的两参数形式相同，只是参数的顺序相反。

    ```sql
    mysql> SELECT INSTR('foobarbar', 'bar');
     -> 4
    mysql> SELECT INSTR('xbar', 'foobar');
     -> 0
    ```

    此函数支持多字节字符，仅在至少一个参数为二进制字符串时区分大小写。如果任一参数为`NULL`，此函数返回`NULL`。

+   `LCASE(*`str`*)`

    `LCASE()` 是 `LOWER()` 的同义词。

    在视图中使用的`LCASE()`在存储视图定义时会被重写为`LOWER()`。（Bug #12844279）

+   `LEFT(*`str`*,*`len`*)`

    返回字符串*`str`*中最左边的*`len`*个字符，如果任何参数为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT LEFT('foobarbar', 5);
     -> 'fooba'
    ```

    此函数支持多字节字符。

+   `LENGTH(*`str`*)`

    返回字符串*`str`*的长度，以字节为单位。多字节字符计为多个字节。这意味着对于包含五个 2 字节字符的字符串，`LENGTH()` 返回`10`，而`CHAR_LENGTH()` 返回`5`。如果*`str`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT LENGTH('text');
     -> 4
    ```

    注意

    `Length()` OpenGIS 空间函数在 MySQL 中被命名为`ST_Length()`。

+   `LOAD_FILE(*`file_name`*)`

    读取文件并将文件内容作为字符串返回。要使用此函数，文件必须位于服务器主机上，必须指定文件的完整路径名，并且必须具有`FILE`权限。文件必须可被服务器读取，且其大小必须小于`max_allowed_packet`字节。如果`secure_file_priv`系统变量设置为非空目录名称，则要加载的文件必须位于该目录中。（在 MySQL 8.0.17 之前，文件必须可被所有人读取，而不仅仅是服务器可读取。）

    如果文件不存在或由于不满足前述条件之一而无法读取，则函数返回`NULL`。

    `character_set_filesystem`系统变量控制给定为文字字符串的文件名的解释。

    ```sql
    mysql> UPDATE t
                SET blob_col=LOAD_FILE('/tmp/picture')
                WHERE id=1;
    ```

+   `LOCATE(*`substr`*,*`str`*)`, `LOCATE(*`substr`*,*`str`*,*`pos`*)`

    第一个语法返回子字符串*`substr`*在字符串*`str`*中第一次出现的位置。第二个语法返回子字符串*`substr`*在字符串*`str`*中从位置*`pos`*开始第一次出现的位置。如果*`str`*中没有*`substr`*，则返回`0`。如果任何参数为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT LOCATE('bar', 'foobarbar');
     -> 4
    mysql> SELECT LOCATE('xbar', 'foobar');
     -> 0
    mysql> SELECT LOCATE('bar', 'foobarbar', 5);
     -> 7
    ```

    此函数是多字节安全的，仅在至少一个参数为二进制字符串时区分大小写。

+   `LOWER(*`str`*)`

    返回根据当前字符集映射将所有字符更改为小写的字符串*`str`*，如果*`str`*为`NULL`，则返回`NULL`。默认字符集为`utf8mb4`。

    ```sql
    mysql> SELECT LOWER('QUADRATICALLY');
     -> 'quadratically'
    ```

    当应用于二进制字符串（`BINARY`, `VARBINARY`, `BLOB`）时，`LOWER()`（和`UPPER()`）是无效的。要对二进制字符串执行大小写转换，首先使用适合存储在字符串中的数据的字符集将其转换为非二进制字符串：

    ```sql
    mysql> SET @str = BINARY 'New York';
    mysql> SELECT LOWER(@str), LOWER(CONVERT(@str USING utf8mb4));
    +-------------+------------------------------------+
    | LOWER(@str) | LOWER(CONVERT(@str USING utf8mb4)) |
    +-------------+------------------------------------+
    | New York    | new york                           |
    +-------------+------------------------------------+
    ```

    对于 Unicode 字符集的排序规则，如果在排序规则名称中指定了 Unicode 排序算法（UCA）版本，则`LOWER()`和`UPPER()`将按照该版本的 UCA 工作，如果未指定版本，则按照 UCA 4.0.0 工作。例如，`utf8mb4_0900_ai_ci`和`utf8mb3_unicode_520_ci`分别按照 UCA 9.0.0 和 5.2.0 工作，而`utf8mb3_unicode_ci`按照 UCA 4.0.0 工作。参见第 12.10.1 节，“Unicode 字符集”。

    此函数是多字节安全的。

    在视图中使用的`LCASE()`被重写为`LOWER()`。

+   `LPAD(*`str`*,*`len`*,*`padstr`*)`

    返回左填充了字符串*`padstr`*到长度为*`len`*字符的字符串*`str`*。如果*`str`*比*`len`*长，则返回值被缩短为*`len`*个字符。

    ```sql
    mysql> SELECT LPAD('hi',4,'??');
     -> '??hi'
    mysql> SELECT LPAD('hi',1,'??');
     -> 'h'
    ```

    如果其任何参数为`NULL`，则返回`NULL`。

+   `LTRIM(*`str`*)`

    返回去除前导空格字符的字符串*`str`*。如果*`str`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT LTRIM('  barbar');
     -> 'barbar'
    ```

    此函数是多字节安全的。

+   `MAKE_SET(*`bits`*,*`str1`*,*`str2`*,...)`

    返回一个集合值（一个包含由`,`字符分隔的子字符串的字符串），其中包含具有对应位在*`bits`*中设置的字符串。*`str1`*对应于位 0，*`str2`*对应于位 1，依此类推。*`str1`*、*`str2`*、`...`中的`NULL`值不会附加到结果中。

    ```sql
    mysql> SELECT MAKE_SET(1,'a','b','c');
     -> 'a'
    mysql> SELECT MAKE_SET(1 | 4,'hello','nice','world');
     -> 'hello,world'
    mysql> SELECT MAKE_SET(1 | 4,'hello','nice',NULL,'world');
     -> 'hello'
    mysql> SELECT MAKE_SET(0,'a','b','c');
     -> ''
    ```

+   `MID(*`str`*,*`pos`*,*`len`*)`

    `MID(*`str`*,*`pos`*,*`len`*)`是`SUBSTRING(*`str`*,*`pos`*,*`len`*)`的同义词。

+   `OCT(*`N`*)`

    返回*`N`*的八进制值的字符串表示，其中*`N`*是一个长长整型(`BIGINT`)数字。这等同于`CONV(*`N`*,10,8)`。如果*`N`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT OCT(12);
     -> '14'
    ```

+   `OCTET_LENGTH(*`str`*)`

    `OCTET_LENGTH()`是`LENGTH()`的同义词。

+   `ORD(*`str`*)`

    如果字符串*`str`*的最左边字符是一个多字节字符，则返回该字符的代码，使用以下公式从其组成字节的数值计算：

    ```sql
     (1st byte code)
    + (2nd byte code * 256)
    + (3rd byte code * 256²) ...
    ```

    如果最左边的字符不是一个多字节字符，`ORD()`返回与`ASCII()`函数相同的值。如果*`str`*为`NULL`，则函数返回`NULL`。

    ```sql
    mysql> SELECT ORD('2');
     -> 50
    ```

+   `POSITION(*`substr`* IN *`str`*)`

    `POSITION(*`substr`* IN *`str`*)`是`LOCATE(*`substr`*,*`str`*)`的同义词。

+   `QUOTE(*`str`*)`

    引用一个字符串以生成一个结果，该结果可以作为 SQL 语句中的正确转义数据值使用。返回的字符串用单引号括起来，并且每个反斜杠(`\`)、单引号(`'`)、ASCII `NUL`和 Control+Z 的实例前面都有一个反斜杠。如果参数为`NULL`，返回值是不用单引号括起来的单词`NULL`。

    ```sql
    mysql> SELECT QUOTE('Don\'t!');
     -> 'Don\'t!'
    mysql> SELECT QUOTE(NULL);
     -> NULL
    ```

    有关比较，请参阅第 11.1.1 节，“字符串文字”中的文字引用规则以及 C API 中的 mysql_real_escape_string_quote()。

+   `REPEAT(*`str`*,*`count`*)`

    返回由字符串*`str`*重复*`count`*次组成的字符串。如果*`count`*小于 1，则返回空字符串。如果*`str`*或*`count`*是`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT REPEAT('MySQL', 3);
     -> 'MySQLMySQLMySQL'
    ```

+   `REPLACE(*`str`*,*`from_str`*,*`to_str`*)`

    返回将所有出现的字符串*`from_str`*替换为字符串*`to_str`*的字符串*`str`*。在搜索*`from_str`*时，`REPLACE()`执行区分大小写的匹配。

    ```sql
    mysql> SELECT REPLACE('www.mysql.com', 'w', 'Ww');
     -> 'WwWwWw.mysql.com'
    ```

    这个函数是多字节安全的。如果任何参数是`NULL`，则返回`NULL`。

+   `REVERSE(*`str`*)`

    返回字符串*`str`*的字符顺序颠倒，如果*`str`*是`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT REVERSE('abc');
     -> 'cba'
    ```

    这个函数是多字节安全的。

+   `RIGHT(*`str`*,*`len`*)`

    返回字符串*`str`*中最右边的*`len`*个字符，如果任何参数���`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT RIGHT('foobarbar', 4);
     -> 'rbar'
    ```

    这个函数是多字节安全的。

+   `RPAD(*`str`*,*`len`*,*`padstr`*)`

    返回字符串*`str`*，右侧用字符串*`padstr`*填充到长度为*`len`*个字符。如果*`str`*比*`len`*长，则返回值被缩短为*`len`*个字符。如果*`str`*、*`padstr`*或*`len`*是`NULL`，则函数返回`NULL`。

    ```sql
    mysql> SELECT RPAD('hi',5,'?');
     -> 'hi???'
    mysql> SELECT RPAD('hi',1,'?');
     -> 'h'
    ```

    这个函数是多字节安全的。

+   `RTRIM(*`str`*)`

    返回删除尾随空格字符的字符串*`str`*。

    ```sql
    mysql> SELECT RTRIM('barbar   ');
     -> 'barbar'
    ```

    这个函数是多字节安全的，如果*`str`*是`NULL`，则返回`NULL`。

+   `SOUNDEX(*`str`*)`

    从*`str`*返回一个 soundex 字符串，如果*`str`*是`NULL`，则返回`NULL`。两个听起来几乎相同的字符串应该具有相同的 soundex 字符串。标准的 soundex 字符串长度为四个字符，但`SOUNDEX()`函数返回一个任意长的字符串。您可以对结果使用`SUBSTRING()`来获取标准的 soundex 字符串。*`str`*中的所有非字母字符都将被忽略。所有 A-Z 范围之外的国际字母字符都被视为元音。

    重要

    在使用`SOUNDEX()`时，您应该注意以下限制：

    +   此函数目前的实现仅适用于英语语言的字符串。其他语言的字符串可能产生不可靠的结果。

    +   不能保证此函数与使用多字节字符集（包括`utf-8`）的字符串提供一致的结果。有关更多信息，请参见 Bug #22638。

    ```sql
    mysql> SELECT SOUNDEX('Hello');
     -> 'H400'
    mysql> SELECT SOUNDEX('Quadratically');
     -> 'Q36324'
    ```

    注意

    此函数实现原始 Soundex 算法，而不是更流行的增强版本（也由 D. Knuth 描述）。区别在于原始版本首先丢弃元音，其次是重复字符，而增强版本首先丢弃重复字符，其次是元音。

+   `*`expr1`* SOUNDS LIKE *`expr2`*`

    这与`SOUNDEX(*`expr1`*) = SOUNDEX(*`expr2`*)`相同。

+   `SPACE(*`N`*)`

    返回由*`N`*个空格字符组成的字符串，如果*`N`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT SPACE(6);
     -> '      '
    ```

+   `SUBSTR(*`str`*,*`pos`*)`, `SUBSTR(*`str`* FROM *`pos`*)`, `SUBSTR(*`str`*,*`pos`*,*`len`*)`, `SUBSTR(*`str`* FROM *`pos`* FOR *`len`*)`

    `SUBSTR()`是`SUBSTRING()`的同义词。

+   `SUBSTRING(*`str`*,*`pos`*)`, `SUBSTRING(*`str`* FROM *`pos`*)`, `SUBSTRING(*`str`*,*`pos`*,*`len`*)`, `SUBSTRING(*`str`* FROM *`pos`* FOR *`len`*)`

    没有*`len`*参数的形式返回从位置*`pos`*开始的字符串*`str`*的子字符串。有*`len`*参数的形式返回从位置*`pos`*开始，长度为*`len`*的子字符串。使用`FROM`的形式是标准的 SQL 语法。也可以对*`pos`*使用负值。在这种情况下，子字符串的开头是距离字符串末尾*`pos`*个字符，而不是距离开头。在这个函数的任何形式中，*`pos`*都可以使用负值。*`pos`*的值为 0 时返回空字符串。

    对于`SUBSTRING()`的所有形式，要提取子字符串的字符串中第一个字符的位置被认为是`1`。

    ```sql
    mysql> SELECT SUBSTRING('Quadratically',5);
     -> 'ratically'
    mysql> SELECT SUBSTRING('foobarbar' FROM 4);
     -> 'barbar'
    mysql> SELECT SUBSTRING('Quadratically',5,6);
     -> 'ratica'
    mysql> SELECT SUBSTRING('Sakila', -3);
     -> 'ila'
    mysql> SELECT SUBSTRING('Sakila', -5, 3);
     -> 'aki'
    mysql> SELECT SUBSTRING('Sakila' FROM -4 FOR 2);
     -> 'ki'
    ```

    此函数是多字节安全的。如果其任何参数为`NULL`，则返回`NULL`。

    如果*`len`*小于 1，则结果为空字符串。

+   `SUBSTRING_INDEX(*`str`*,*`delim`*,*`count`*)`

    返回字符串*`str`*中在定界符*`delim`*出现*`count`*次之前的子字符串。如果*`count`*为正数，则返回最终定界符（从左边计数）左侧的所有内容。如果*`count`*为负数，则返回最终定界符（从右边计数）右侧的所有内容。在搜索*`delim`*时，`SUBSTRING_INDEX()`执行区分大小写的匹配。

    ```sql
    mysql> SELECT SUBSTRING_INDEX('www.mysql.com', '.', 2);
     -> 'www.mysql'
    mysql> SELECT SUBSTRING_INDEX('www.mysql.com', '.', -2);
     -> 'mysql.com'
    ```

    此函数是多字节安全的。

    如果`SUBSTRING_INDEX()`的任何参数为`NULL`，则返回`NULL`。

+   `TO_BASE64(*`str`*)`

    将字符串参数转换为 base-64 编码形式，并将结果作为具有连接字符集和排序规则的字符字符串返回。如果参数不是字符串，则在进行转换之前将其转换为字符串。如果参数为`NULL`，则结果为`NULL`。可以使用`FROM_BASE64()`函数解码 base-64 编码的字符串。

    ```sql
    mysql> SELECT TO_BASE64('abc'), FROM_BASE64(TO_BASE64('abc'));
     -> 'JWJj', 'abc'
    ```

    存在不同的 base-64 编码方案。这些是`TO_BASE64()`和`FROM_BASE64()`使用的编码和解码规则：

    +   字母值为 62 的编码为`'+'`。

    +   字母值为 63 的编码为`'/'`。

    +   编码输出由 4 个可打印字符组成。每 3 个字节的输入数据使用 4 个字符进行编码。如果最后一组不完整，则使用`'='`字符填充至长度为 4。

    +   每 76 个编码输出字符后添加一个换行符，将长输出分成多行。

    +   解码识别并忽略换行符、回车符、制表符和空格。

+   [`TRIM([{BOTH | LEADING | TRAILING} [*`remstr`*] FROM] *`str`*)`](string-functions.html#function_trim), [`TRIM([*`remstr`* FROM] *`str`*)`](string-functions.html#function_trim)

    返回去除所有*`remstr`*前缀或后缀的字符串*`str`*。如果未给出`BOTH`、`LEADING`或`TRAILING`中的任何一个标识符，则假定为`BOTH`。*`remstr`*是可选的，如果未指定，则删除空格。

    ```sql
    mysql> SELECT TRIM('  bar   ');
     -> 'bar'
    mysql> SELECT TRIM(LEADING 'x' FROM 'xxxbarxxx');
     -> 'barxxx'
    mysql> SELECT TRIM(BOTH 'x' FROM 'xxxbarxxx');
     -> 'bar'
    mysql> SELECT TRIM(TRAILING 'xyz' FROM 'barxxyz');
     -> 'barx'
    ```

    此函数是多字节安全的。如果其任何参数为`NULL`，则返回`NULL`。

+   `UCASE(*`str`*)`

    `UCASE()`是`UPPER()`的同义词。

    在视图中使用的`UCASE()`会被重写为`UPPER()`。

+   `UNHEX(*`str`*)`

    对于字符串参数*`str`*，`UNHEX(*`str`*)`将参数中的每对字符解释为十六进制数，并将其转换为由该数字表示的字节。返回值是一个二进制字符串。

    ```sql
    mysql> SELECT UNHEX('4D7953514C');
     -> 'MySQL'
    mysql> SELECT X'4D7953514C';
     -> 'MySQL'
    mysql> SELECT UNHEX(HEX('string'));
     -> 'string'
    mysql> SELECT HEX(UNHEX('1267'));
     -> '1267'
    ```

    参数字符串中的字符必须是合法的十六进制数字：`'0'` .. `'9'`，`'A'` .. `'F'`，`'a'` .. `'f'`。如果参数包含任何非十六进制数字，或者本身为`NULL`，则结果为`NULL`：

    ```sql
    mysql> SELECT UNHEX('GG');
    +-------------+
    | UNHEX('GG') |
    +-------------+
    | NULL        |
    +-------------+

    mysql> SELECT UNHEX(NULL);
    +-------------+
    | UNHEX(NULL) |
    +-------------+
    | NULL        |
    +-------------+
    ```

    如果`UNHEX()`的参数是`BINARY`列，则还可能出现`NULL`结果，因为存储时值会使用`0x00`字节进行填充，但在检索时这些字节不会被去除。例如，`'41'`被存储到`CHAR(3)`列中为`'41 '`，检索时为`'41'`（去除了尾随填充空格），因此列值的`UNHEX()`返回`X'41'`。相比之下，`'41'`被存储到`BINARY(3)`列中为`'41\0'`，检索时为`'41\0'`（尾随填充`0x00`字节未被去除）。`'\0'`不是合法的十六进制数字，因此列值的`UNHEX()`返回`NULL`。

    对于数值参数*`N`*，`UNHEX()`不执行`HEX(*`N`*)`的逆操作。请改用`CONV(HEX(*`N`*),16,10)`。请参阅`HEX()`的描述。

    如果在**mysql**客户端中调用`UNHEX()`，二进制字符串将以十六进制表示形式显示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参阅第 6.5.1 节，“mysql — MySQL 命令行客户端”。

+   `UPPER(*`str`*)`

    返回根据当前字符集映射将所有字符更改为大写的字符串*`str`*，如果*`str`*为`NULL`则返回`NULL`。默认字符集为`utf8mb4`。

    ```sql
    mysql> SELECT UPPER('Hej');
     -> 'HEJ'
    ```

    有关如何对二进制字符串（`BINARY`, `VARBINARY`, `BLOB`）执行大小写转换以及有关 Unicode 字符集的大小写折叠的信息，请参阅`LOWER()`的描述，这些信息也适用于`UPPER()`。

    此函数支持多字节。

    在视图中使用的`UCASE()`将被重写为`UPPER()`。

+   [`WEIGHT_STRING(*`str`* [AS {CHAR|BINARY}(*`N`*)] [*`flags`*])`](string-functions.html#function_weight-string)

    此函数返回输入字符串的权重字符串。返回值是一个二进制字符串，表示字符串的比较和排序值，如果参数为`NULL`则返回`NULL`。具有以下属性：

    +   如果 `WEIGHT_STRING(*`str1`*)` = `WEIGHT_STRING(*`str2`*)`，那么 `*`str1`* = *`str2`*`（*`str1`* 和 *`str2`* 被视为相等）

    +   如果 `WEIGHT_STRING(*`str1`*)` < `WEIGHT_STRING(*`str2`*)`，那么 `*`str1`* < *`str2`*`（*`str1`* 在 *`str2`* 之前排序）

    `WEIGHT_STRING()` 是一个用于内部调试的函数。其行为可能会在 MySQL 版本之间发生变化而无需通知。它可用于测试和调试排序规则，特别是在添加新的排序规则时。参见 第 12.14 节，“向字符集添加排序规则”。

    这个列表简要总结了参数。更多细节在列表后的讨论中给出。

    +   *`str`*：输入字符串表达式。

    +   `AS` 子句：可选；将输入字符串转换为给定类型和长度。

    +   *`flags`*：可选；未使用。

    输入字符串 *`str`* 是一个字符串表达式。如果输入是非二进制（字符）字符串，如 `CHAR`、`VARCHAR` 或 `TEXT` 值，则返回值包含字符串的排序规则权重。如果输入是二进制（字节）字符串，如 `BINARY`、`VARBINARY` 或 `BLOB` 值，则返回值与输入相同（二进制字符串中每个字节的权重是字节值）。如果输入为 `NULL`，`WEIGHT_STRING()` 返回 `NULL`。

    示例：

    ```sql
    mysql> SET @s = _utf8mb4 'AB' COLLATE utf8mb4_0900_ai_ci;
    mysql> SELECT @s, HEX(@s), HEX(WEIGHT_STRING(@s));
    +------+---------+------------------------+
    | @s   | HEX(@s) | HEX(WEIGHT_STRING(@s)) |
    +------+---------+------------------------+
    | AB   | 4142    | 1C471C60               |
    +------+---------+------------------------+
    ```

    ```sql
    mysql> SET @s = _utf8mb4 'ab' COLLATE utf8mb4_0900_ai_ci;
    mysql> SELECT @s, HEX(@s), HEX(WEIGHT_STRING(@s));
    +------+---------+------------------------+
    | @s   | HEX(@s) | HEX(WEIGHT_STRING(@s)) |
    +------+---------+------------------------+
    | ab   | 6162    | 1C471C60               |
    +------+---------+------------------------+
    ```

    ```sql
    mysql> SET @s = CAST('AB' AS BINARY);
    mysql> SELECT @s, HEX(@s), HEX(WEIGHT_STRING(@s));
    +------+---------+------------------------+
    | @s   | HEX(@s) | HEX(WEIGHT_STRING(@s)) |
    +------+---------+------------------------+
    | AB   | 4142    | 4142                   |
    +------+---------+------------------------+
    ```

    ```sql
    mysql> SET @s = CAST('ab' AS BINARY);
    mysql> SELECT @s, HEX(@s), HEX(WEIGHT_STRING(@s));
    +------+---------+------------------------+
    | @s   | HEX(@s) | HEX(WEIGHT_STRING(@s)) |
    +------+---------+------------------------+
    | ab   | 6162    | 6162                   |
    +------+---------+------------------------+
    ```

    前面的示例使用 `HEX()` 来显示 `WEIGHT_STRING()` 的结果。因为结果是一个二进制值，当结果包含不可打印的值时，`HEX()` 尤其有用，可以将其显示为可打印形式：

    ```sql
    mysql> SET @s = CONVERT(X'C39F' USING utf8mb4) COLLATE utf8mb4_czech_ci;
    mysql> SELECT HEX(WEIGHT_STRING(@s));
    +------------------------+
    | HEX(WEIGHT_STRING(@s)) |
    +------------------------+
    | 0FEA0FEA               |
    +------------------------+
    ```

    对于非`NULL`的返回值，如果值的长度在 `VARBINARY` 的最大长度内，则值的数据类型为 `VARBINARY`，否则数据类型为 `BLOB`。

    `AS` 子句可以用于将输入字符串转换为非二进制或二进制字符串，并强制其达到给定长度：

    +   `AS CHAR(*`N`*)` 将字符串转换为非二进制字符串，并在右侧用空格填充到长度为 *`N`* 个字符。*`N`* 必须至少为 1。如果 *`N`* 小于输入字符串的长度，则字符串将被截断为 *`N`* 个字符。截断不会发出警告。

    +   `AS BINARY(*`N`*)` 类似，但将字符串转换为二进制字符串，*`N`* 以字节为单位（而非字符），填充使用 `0x00` 字节（而非空格）。

    ```sql
    mysql> SET NAMES 'latin1';
    mysql> SELECT HEX(WEIGHT_STRING('ab' AS CHAR(4)));
    +-------------------------------------+
    | HEX(WEIGHT_STRING('ab' AS CHAR(4))) |
    +-------------------------------------+
    | 41422020                            |
    +-------------------------------------+
    mysql> SET NAMES 'utf8mb4';
    mysql> SELECT HEX(WEIGHT_STRING('ab' AS CHAR(4)));
    +-------------------------------------+
    | HEX(WEIGHT_STRING('ab' AS CHAR(4))) |
    +-------------------------------------+
    | 1C471C60                            |
    +-------------------------------------+
    ```

    ```sql
    mysql> SELECT HEX(WEIGHT_STRING('ab' AS BINARY(4)));
    +---------------------------------------+
    | HEX(WEIGHT_STRING('ab' AS BINARY(4))) |
    +---------------------------------------+
    | 61620000                              |
    +---------------------------------------+
    ```

    *`flags`* 子句目前未被使用。

    如果在**mysql**客户端中调用`WEIGHT_STRING()`，二进制字符串将以十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参见第 6.5.1 节，“mysql — The MySQL Command-Line Client”。
