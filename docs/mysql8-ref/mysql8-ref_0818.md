# 14.12 位函数和运算符

> 原文：[`dev.mysql.com/doc/refman/8.0/en/bit-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html)

**表 14.17 位函数和运算符**

| 名称 | 描述 |
| --- | --- |
| `&` | 按位与 |
| `>>` | 右移 |
| `<<` | 左移 |
| `^` | 按位异或 |
| `BIT_COUNT()` | 返回设置的位数 |
| `&#124;` | 按位或 |
| `~` | 按位取反 |

以下列表描述了可用的位函数和运算符：

+   `|`

    按位或。

    结果类型取决于参数是作为二进制字符串还是数字进行评估：

    +   当参数具有二进制字符串类型且至少有一个不是十六进制文字、位文字或`NULL`文字时，进行二进制字符串运算。否则进行数值运算，必要时将参数转换为无符号 64 位整数。

    +   二进制字符串运算会产生与参数相同长度的二进制字符串。如果参数长度不相等，则会出现`ER_INVALID_BITWISE_OPERANDS_SIZE`错误。数值运算会产生一个无符号 64 位整数。

    更多信息，请参阅本节的介绍性讨论。

    ```sql
    mysql> SELECT 29 | 15;
     -> 31
    mysql> SELECT _binary X'40404040' | X'01020304';
     -> 'ABCD'
    ```

    如果在**mysql**客户端中调用按位或运算，二进制字符串结果将以十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参阅 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

+   `&`

    按位与。

    结果类型取决于参数是作为二进制字符串还是数字进行评估：

    +   当参数具有二进制字符串类型且至少有一个不是十六进制文字、位文字或`NULL`文字时，进行二进制字符串运算。否则进行数值运算，必要时将参数转换为无符号 64 位整数。

    +   二进制字符串运算会产生与参数相同长度的二进制字符串。如果参数长度不相等，则会出现`ER_INVALID_BITWISE_OPERANDS_SIZE`错误。数值运算会产生一个无符号 64 位整数。

    更多信息，请参阅本节的介绍性讨论。

    ```sql
    mysql> SELECT 29 & 15;
     -> 13
    mysql> SELECT HEX(_binary X'FF' & b'11110000');
     -> 'F0'
    ```

    如果在**mysql**客户端中调用按位与，二进制字符串结果将以十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参阅第 6.5.1 节，“mysql — MySQL 命令行客户端”。

+   `^`

    按位异或。

    结果类型取决于参数是作为二进制字符串还是数字进行评估：

    +   当参数具有二进制字符串类型且至少一个参数不是十六进制文字、比特文字或`NULL`文字时，进行二进制字符串运算。否则进行数值运算，并根据需要将参数转换为无符号 64 位整数。

    +   二进制字符串运算产生与参数长度相同的二进制字符串。如果参数长度不相等，则会出现`ER_INVALID_BITWISE_OPERANDS_SIZE`错误。数值运算产生一个无符号 64 位整数。

    更多信息，请参阅本节的介绍性讨论。

    ```sql
    mysql> SELECT 1 ^ 1;
     -> 0
    mysql> SELECT 1 ^ 0;
     -> 1
    mysql> SELECT 11 ^ 3;
     -> 8
    mysql> SELECT HEX(_binary X'FEDC' ^ X'1111');
     -> 'EFCD'
    ```

    如果在**mysql**客户端中调用按位异或，二进制字符串结果将以十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参阅第 6.5.1 节，“mysql — MySQL 命令行客户端”。

+   `<<`

    将一个长整型（`BIGINT`）数字或二进制字符串向左移位。

    结果类型取决于比特参数是作为二进制字符串还是数字进行评估：

    +   当比特参数具有二进制字符串类型且不是十六进制文字、比特文字或`NULL`文字时，进行二进制字符串运算。否则进行数值运算，并根据需要将参数转换为无符号 64 位整数。

    +   二进制字符串运算产生与比特参数长度相同的二进制字符串。数值运算产生一个无符号 64 位整数。

    超出值末尾的位将被丢弃，不会有警告，无论参数类型如何。特别是，如果移位计数大于或等于比特参数中的位数，则结果中的所有位都为 0。

    更多信息，请参阅本节的介绍性讨论。

    ```sql
    mysql> SELECT 1 << 2;
     -> 4
    mysql> SELECT HEX(_binary X'00FF00FF00FF' << 8);
     -> 'FF00FF00FF00'
    ```

    如果在**mysql**客户端内调用位移操作，则根据`--binary-as-hex`的值，二进制字符串结果将以十六进制表示。有关该选项的更多信息，请参阅 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

+   `>>`

    将一个长整型(`BIGINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT"))数字或二进制字符串向右移动。

    结果类型取决于位参数是作为二进制字符串还是数字进行评估：

    +   当位参数具有二进制字符串类型且不是十六进制文字、位文字或`NULL`文字时，进行二进制字符串评估。否则进行数值评估，必要时将参数转换为无符号 64 位整数。

    +   二进制字符串评估会产生与位参数相同长度的二进制字符串。数值评估会产生一个无符号 64 位整数。

    无论参数类型如何，超出值末尾的位都会被丢弃而不发出警告。特别是，如果位移计数大于或等于位参数中的位数，则结果中的所有位都为 0。

    有关更多信息，请参阅本节中的介绍性讨论。

    ```sql
    mysql> SELECT 4 >> 2;
     -> 1
    mysql> SELECT HEX(_binary X'00FF00FF00FF' >> 8);
     -> '0000FF00FF00'
    ```

    如果在**mysql**客户端内调用位移操作，则根据`--binary-as-hex`的值，二进制字符串结果将以十六进制表示。有关该选项的更多信息，请参阅 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

+   `~`

    反转所有位。

    结果类型取决于位参数是作为二进制字符串还是数字进行评估：

    +   当位参数具有二进制字符串类型且不是十六进制文字、位文字或`NULL`文字时，进行二进制字符串评估。否则进行数值评估，必要时将参数转换为无符号 64 位整数。

    +   二进制字符串评估会产生与位参数相同长度的二进制字符串。数值评估会产生一个无符号 64 位整数。

    有关更多信息，请参阅本节中的介绍性讨论。

    ```sql
    mysql> SELECT 5 & ~1;
     -> 4
    mysql> SELECT HEX(~X'0000FFFF1111EEEE');
     -> 'FFFF0000EEEE1111'
    ```

    如果在**mysql**客户端内调用位求反操作，则根据`--binary-as-hex`的值，二进制字符串结果将以十六进制表示。有关该选项的更多信息，请参阅 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

+   `BIT_COUNT(*`N`*)`

    返回参数 *`N`* 中设置的位数作为无符号 64 位整数，如果参数为 `NULL` 则返回 `NULL`。

    ```sql
    mysql> SELECT BIT_COUNT(64), BIT_COUNT(BINARY 64);
     -> 1, 7
    mysql> SELECT BIT_COUNT('64'), BIT_COUNT(_binary '64');
     -> 1, 7
    mysql> SELECT BIT_COUNT(X'40'), BIT_COUNT(_binary X'40');
     -> 1, 1
    ```

位函数和运算符包括 `BIT_COUNT()`, `BIT_AND()`, `BIT_OR()`, `BIT_XOR()`, `&`, `|`, `^`, `~`, `<<`, 以及 `>>`。（`BIT_AND()`, `BIT_OR()`, 和 `BIT_XOR()` 聚合函数在 第 14.19.1 节，“聚合函数描述” 中有描述。）在 MySQL 8.0 之前，位函数和运算符需要 `BIGINT`（64 位整数）参数，并返回 `BIGINT` 值，因此它们的最大范围为 64 位。非 `BIGINT` 参数在执行操作之前被转换为 `BIGINT`，并且可能发生截断。

在 MySQL 8.0 中，位函数和运算符允许二进制字符串类型的参数（`BINARY`, `VARBINARY`, 以及 `BLOB` 类型），并返回相同类型的值，这使它们能够接受参数并生成大于 64 位的返回值。非二进制字符串参数被转换为 `BIGINT` 并按照此类处理，就像以前一样。

这种行为变化的一个影响是，在 MySQL 8.0 中对二进制字符串参数进行位操作可能会产生与 5.7 中不同的结果。有关如何在 MySQL 5.7 中准备可能的 MySQL 5.7 和 8.0 之间不兼容性的信息，请参阅 位函数和运算符，在 MySQL 5.7 参考手册 中。

+   MySQL 8.0 之前的位操作

+   MySQL 8.0 中的位操作

+   二进制字符串位操作示例

+   位与、或和异或操作

+   位取反和移位操作

+   BIT_COUNT() 操作 操作")

+   BIT_AND(), BIT_OR(), 和 BIT_XOR() 操作, BIT_OR(), 和 BIT_XOR() 操作")

+   十六进制文字面量、位文字面量和 NULL 文字面量的特殊处理

+   与 MySQL 5.7 不兼容的位操作

### MySQL 8.0 之前的位操作

MySQL 8.0 之前的位操作仅处理无符号 64 位整数参数和结果值（即无符号`BIGINT` 值）。必要时将其他类型的参数转换为`BIGINT`。示例：

+   此语句操作数字文字面量，将其视为无符号 64 位整数：

    ```sql
    mysql> SELECT 127 | 128, 128 << 2, BIT_COUNT(15);
    +-----------+----------+---------------+
    | 127 | 128 | 128 << 2 | BIT_COUNT(15) |
    +-----------+----------+---------------+
    |       255 |      512 |             4 |
    +-----------+----------+---------------+
    ```

+   在执行此语句之前，对字符串参数进行了数字转换（例如，`'127'` 转换为 `127`），然后执行与第一个语句相同的操作并产生相同的结果：

    ```sql
    mysql> SELECT '127' | '128', '128' << 2, BIT_COUNT('15');
    +---------------+------------+-----------------+
    | '127' | '128' | '128' << 2 | BIT_COUNT('15') |
    +---------------+------------+-----------------+
    |           255 |        512 |               4 |
    +---------------+------------+-----------------+
    ```

+   此语句使用十六进制文字面量作为位操作参数。MySQL 默认将十六进制文字面量视为二进制字符串，但在数字上下文中将其评估为数字（参见第 11.1.4 节，“十六进制文字面量”）。在 MySQL 8.0 之前，数字上下文包括位操作。示例：

    ```sql
    mysql> SELECT X'7F' | X'80', X'80' << 2, BIT_COUNT(X'0F');
    +---------------+------------+------------------+
    | X'7F' | X'80' | X'80' << 2 | BIT_COUNT(X'0F') |
    +---------------+------------+------------------+
    |           255 |        512 |                4 |
    +---------------+------------+------------------+
    ```

    在位操作中处理位值文字面量类似于十六进制文字面量（即作为数字）。

### MySQL 8.0 中的位操作

MySQL 8.0 扩展了位操作，直接处理二进制字符串参数（无需转换）并产生二进制字符串结果。（不是整数或二进制字符串的参数仍然会像以前一样转换为整数。）此扩展以以下方式增强了位操作：

+   可以对超过 64 位的值执行位操作。

+   对于更自然地表示为二进制字符串而不是整数的值执行位操作更容易。

例如，考虑 UUID 值和 IPv6 地址，它们具有人类可读的文本格式，如下所示：

```sql
UUID: 6ccd780c-baba-1026-9564-5b8c656024db
IPv6: fe80::219:d1ff:fe91:1a72
```

在这些格式的文本字符串上操作是繁琐的。一个替代方法是将它们转换为没有分隔符的固定长度二进制字符串。`UUID_TO_BIN()`和`INET6_ATON()`分别产生数据类型为`BINARY(16)`的值，一个 16 字节（128 位）长的二进制字符串。以下语句说明了这一点（`HEX()`用于生成可显示的值）：

```sql
mysql> SELECT HEX(UUID_TO_BIN('6ccd780c-baba-1026-9564-5b8c656024db'));
+----------------------------------------------------------+
| HEX(UUID_TO_BIN('6ccd780c-baba-1026-9564-5b8c656024db')) |
+----------------------------------------------------------+
| 6CCD780CBABA102695645B8C656024DB                         |
+----------------------------------------------------------+
mysql> SELECT HEX(INET6_ATON('fe80::219:d1ff:fe91:1a72'));
+---------------------------------------------+
| HEX(INET6_ATON('fe80::219:d1ff:fe91:1a72')) |
+---------------------------------------------+
| FE800000000000000219D1FFFE911A72            |
+---------------------------------------------+
```

那些二进制值可以通过位操作轻松操作，执行诸如从 UUID 值中提取时间戳或从 IPv6 地址中提取网络和主机部分等操作。（有关示例，请参见本讨论后面。）

计为二进制字符串的参数包括列值、例程参数、局部变量和具有二进制字符串类型的用户定义变量：`BINARY`、`VARBINARY`或`BLOB`类型之一。

那么十六进制文字和位文字呢？回想一下，在 MySQL 中，默认情况下这些是二进制字符串，但在数字上下文中是数字。在 MySQL 8.0 中，它们如何处理用于位操作？MySQL 是否继续在数字上下文中评估它们，就像在 MySQL 8.0 之前所做的那样？还是位操作现在将它们作为二进制字符串进行评估，因为二进制字符串可以“原生”处理而无需转换？

答案：通常会使用十六进制文字或位文字指定位操作的参数，以表示数字，因此当所有位参数都是十六进制或位文字时，MySQL 继续在数字上下文中评估位操作，以保持向后兼容性。如果您需要将其评估为二进制字符串，那很容易实现：至少使用一个文字的`_binary`引入者。

+   这些位操作将十六进制文字和位文字作为整数进行评估：

    ```sql
    mysql> SELECT X'40' | X'01', b'11110001' & b'01001111';
    +---------------+---------------------------+
    | X'40' | X'01' | b'11110001' & b'01001111' |
    +---------------+---------------------------+
    |            65 |                        65 |
    +---------------+---------------------------+
    ```

+   这些位操作将十六进制文字和位文字作为二进制字符串进行评估，这是由`_binary`引入者引起的：

    ```sql
    mysql> SELECT _binary X'40' | X'01', b'11110001' & _binary b'01001111';
    +-----------------------+-----------------------------------+
    | _binary X'40' | X'01' | b'11110001' & _binary b'01001111' |
    +-----------------------+-----------------------------------+
    | A                     | A                                 |
    +-----------------------+-----------------------------------+
    ```

尽管两个语句中的位操作都产生数值为 65 的结果，但第二个语句在二进制字符串上下文中运行，65 在 ASCII 中是`A`。

在数字评估上下文中，十六进制文字和位文字参数的允许值最多为 64 位，结果也是如此。相比之下，在二进制字符串评估上下文中，允许的参数（和结果）可以超过 64 位：

```sql
mysql> SELECT _binary X'4040404040404040' | X'0102030405060708';
+---------------------------------------------------+
| _binary X'4040404040404040' | X'0102030405060708' |
+---------------------------------------------------+
| ABCDEFGH                                          |
+---------------------------------------------------+
```

有几种方法可以引用位操作中的十六进制文字或位文字，以导致二进制字符串评估：

```sql
_binary *literal*
BINARY *literal*
CAST(*literal* AS BINARY)
```

将十六进制文字或位文字分配给用户定义变量是产生二进制字符串评估的另一种方法，这将导致具有二进制字符串类型的变量：

```sql
mysql> SET @v1 = X'40', @v2 = X'01', @v3 = b'11110001', @v4 = b'01001111';
mysql> SELECT @v1 | @v2, @v3 & @v4;
+-----------+-----------+
| @v1 | @v2 | @v3 & @v4 |
+-----------+-----------+
| A         | A         |
+-----------+-----------+
```

在二进制字符串上下文中，位操作的参数必须具有相同的长度，否则会出现`ER_INVALID_BITWISE_OPERANDS_SIZE`错误：

```sql
mysql> SELECT _binary X'40' | X'0001';
ERROR 3513 (HY000): Binary operands of bitwise
operators must be of equal length
```

为满足等长要求，使用前导零位填充较短值，或者如果较长值以前导零位开始且可以接受较短结果值，则剥离它们：

```sql
mysql> SELECT _binary X'0040' | X'0001';
+---------------------------+
| _binary X'0040' | X'0001' |
+---------------------------+
|  A                        |
+---------------------------+
mysql> SELECT _binary X'40' | X'01';
+-----------------------+
| _binary X'40' | X'01' |
+-----------------------+
| A                     |
+-----------------------+
```

填充或剥离也可以使用函数来完成，例如`LPAD()`、`RPAD()`、`SUBSTR()`或`CAST()`。在这种情况下，表达式参数不再都是文字，并且`_binary`变得不必要。示例：

```sql
mysql> SELECT LPAD(X'40', 2, X'00') | X'0001';
+---------------------------------+
| LPAD(X'40', 2, X'00') | X'0001' |
+---------------------------------+
|  A                              |
+---------------------------------+
mysql> SELECT X'40' | SUBSTR(X'0001', 2, 1);
+-------------------------------+
| X'40' | SUBSTR(X'0001', 2, 1) |
+-------------------------------+
| A                             |
+-------------------------------+
```

### 二进制字符串位操作示例

以下示例说明了使用位操作来提取 UUID 值的部分，例如时间戳和 IEEE 802 节点号。此技术需要为每个提取部分准备位掩码。

将文本 UUID 转换为相应的 16 字节二进制值，以便在二进制字符串上下文中使用位操作进行操作：

```sql
mysql> SET @uuid = UUID_TO_BIN('6ccd780c-baba-1026-9564-5b8c656024db');
mysql> SELECT HEX(@uuid);
+----------------------------------+
| HEX(@uuid)                       |
+----------------------------------+
| 6CCD780CBABA102695645B8C656024DB |
+----------------------------------+
```

为值的时间戳和节点号部分构造位掩码。时间戳包括前三部分（64 位，位 0 到 63），节点号是最后一部分（48 位，位 80 到 127）：

```sql
mysql> SET @ts_mask = CAST(X'FFFFFFFFFFFFFFFF' AS BINARY(16));
mysql> SET @node_mask = CAST(X'FFFFFFFFFFFF' AS BINARY(16)) >> 80;
mysql> SELECT HEX(@ts_mask);
+----------------------------------+
| HEX(@ts_mask)                    |
+----------------------------------+
| FFFFFFFFFFFFFFFF0000000000000000 |
+----------------------------------+
mysql> SELECT HEX(@node_mask);
+----------------------------------+
| HEX(@node_mask)                  |
+----------------------------------+
| 00000000000000000000FFFFFFFFFFFF |
+----------------------------------+
```

这里使用`CAST(... AS BINARY(16))`函数，因为掩码必须与其应用的 UUID 值长度相同。可以使用其他函数将掩码填充到所需长度以产生相同的结果：

```sql
SET @ts_mask= RPAD(X'FFFFFFFFFFFFFFFF' , 16, X'00');
SET @node_mask = LPAD(X'FFFFFFFFFFFF', 16, X'00') ;
```

使用掩码提取时间戳和节点号部分：

```sql
mysql> SELECT HEX(@uuid & @ts_mask) AS 'timestamp part';
+----------------------------------+
| timestamp part                   |
+----------------------------------+
| 6CCD780CBABA10260000000000000000 |
+----------------------------------+
mysql> SELECT HEX(@uuid & @node_mask) AS 'node part';
+----------------------------------+
| node part                        |
+----------------------------------+
| 000000000000000000005B8C656024DB |
+----------------------------------+
```

前面的示例使用了这些位操作：右移（`>>`)和按位与（`&`)。

注意

`UUID_TO_BIN()`接受一个标志，导致生成的二进制 UUID 值中的一些位重新排列。如果使用该标志，请相应修改提取掩码。

下一个示例使用位操作来提取 IPv6 地址的网络和主机部分。假设网络部分长度为 80 位。那么主机部分长度为 128 − 80 = 48 位。要提取地址的网络和主机部分，将其转换为二进制字符串，然后在二进制字符串上下文中使用位操作。

将文本 IPv6 地址转换为相应的二进制字符串：

```sql
mysql> SET @ip = INET6_ATON('fe80::219:d1ff:fe91:1a72');
```

定义网络长度（以位为单位）：

```sql
mysql> SET @net_len = 80;
```

通过将全为 1 的地址左移或右移来构造网络和主机掩码。为此，从地址`::`开始，这是所有零的简写，可以通过将其转换为二进制字符串来查看：

```sql
mysql> SELECT HEX(INET6_ATON('::')) AS 'all zeros';
+----------------------------------+
| all zeros                        |
+----------------------------------+
| 00000000000000000000000000000000 |
+----------------------------------+
```

要生成补码值（全为 1），使用`~`运算符来反转位：

```sql
mysql> SELECT HEX(~INET6_ATON('::')) AS 'all ones';
+----------------------------------+
| all ones                         |
+----------------------------------+
| FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF |
+----------------------------------+
```

将全为 1 的值左移或右移以生成网络和主机掩码：

```sql
mysql> SET @net_mask = ~INET6_ATON('::') << (128 - @net_len);
mysql> SET @host_mask = ~INET6_ATON('::') >> @net_len;
```

显示掩码以验证其覆盖地址的正确部分：

```sql
mysql> SELECT INET6_NTOA(@net_mask) AS 'network mask';
+----------------------------+
| network mask               |
+----------------------------+
| ffff:ffff:ffff:ffff:ffff:: |
+----------------------------+
mysql> SELECT INET6_NTOA(@host_mask) AS 'host mask';
+------------------------+
| host mask              |
+------------------------+
| ::ffff:255.255.255.255 |
+------------------------+
```

提取并显示地址的网络部分和主机部分：

```sql
mysql> SET @net_part = @ip & @net_mask;
mysql> SET @host_part = @ip & @host_mask;
mysql> SELECT INET6_NTOA(@net_part) AS 'network part';
+-----------------+
| network part    |
+-----------------+
| fe80::219:0:0:0 |
+-----------------+
mysql> SELECT INET6_NTOA(@host_part) AS 'host part';
+------------------+
| host part        |
+------------------+
| ::d1ff:fe91:1a72 |
+------------------+
```

前面的示例使用了这些位操作：补码（`~`）、左移（`<<`）和按位与（`&`）。

剩余的讨论提供了每组位操作的参数处理细节，位操作中字面值处理的更多信息，以及 MySQL 8.0 与旧版 MySQL 之间的潜在不兼容性。

### 按位与、或和异或操作

对于`&`、`|`和`^`位操作，结果类型取决于参数是作为二进制字符串还是数字进行评估：

+   当参数具有二进制字符串类型且至少有一个参数不是十六进制字面值、位字面值或`NULL`字面值时，进行二进制字符串评估。否则进行数值评估，必要时将参数转换为无符号 64 位整数。

+   二进制字符串评估产生与参数相同长度的二进制字符串。如果参数长度不相等，则会出现 `ER_INVALID_BITWISE_OPERANDS_SIZE` 错误。数值评估产生一个无符号 64 位整数。

数值评估的示例：

```sql
mysql> SELECT 64 | 1, X'40' | X'01';
+--------+---------------+
| 64 | 1 | X'40' | X'01' |
+--------+---------------+
|     65 |            65 |
+--------+---------------+
```

二进制字符串评估的示例：

```sql
mysql> SELECT _binary X'40' | X'01';
+-----------------------+
| _binary X'40' | X'01' |
+-----------------------+
| A                     |
+-----------------------+
mysql> SET @var1 = X'40', @var2 = X'01';
mysql> SELECT @var1 | @var2;
+---------------+
| @var1 | @var2 |
+---------------+
| A             |
+---------------+
```

### 按位补码和移位操作

对于`~`、`<<`和`>>`位操作，结果类型取决于位参数是作为二进制字符串还是数字进行评估：

+   当位参数具有二进制字符串类型且不是十六进制字面值、位字面值或`NULL`字面值时，进行二进制字符串评估。否则进行数值评估，必要时将参数转换为无符号 64 位整数。

+   二进制字符串评估产生与位参数相同长度的二进制字符串。数值评估产生一个无符号 64 位整数。

对于移位操作，超出值末尾的位将被丢弃，而不会有警告，无论参数类型如何。特别是，如果移位计数大于或等于位参数中的位数，则结果中的所有位都为 0。

数值评估的示例：

```sql
mysql> SELECT ~0, 64 << 2, X'40' << 2;
+----------------------+---------+------------+
| ~0                   | 64 << 2 | X'40' << 2 |
+----------------------+---------+------------+
| 18446744073709551615 |     256 |        256 |
+----------------------+---------+------------+
```

二进制字符串评估的示例：

```sql
mysql> SELECT HEX(_binary X'1111000022220000' >> 16);
+----------------------------------------+
| HEX(_binary X'1111000022220000' >> 16) |
+----------------------------------------+
| 0000111100002222                       |
+----------------------------------------+
mysql> SELECT HEX(_binary X'1111000022220000' << 16);
+----------------------------------------+
| HEX(_binary X'1111000022220000' << 16) |
+----------------------------------------+
| 0000222200000000                       |
+----------------------------------------+
mysql> SET @var1 = X'F0F0F0F0';
mysql> SELECT HEX(~@var1);
+-------------+
| HEX(~@var1) |
+-------------+
| 0F0F0F0F    |
+-------------+
```

### BIT_COUNT() 操作

`BIT_COUNT()` 函数始终返回一个无符号 64 位整数，如果参数为`NULL`，则返回`NULL`。

```sql
mysql> SELECT BIT_COUNT(127);
+----------------+
| BIT_COUNT(127) |
+----------------+
|              7 |
+----------------+
mysql> SELECT BIT_COUNT(b'010101'), BIT_COUNT(_binary b'010101');
+----------------------+------------------------------+
| BIT_COUNT(b'010101') | BIT_COUNT(_binary b'010101') |
+----------------------+------------------------------+
|                    3 |                            3 |
+----------------------+------------------------------+
```

### BIT_AND()、BIT_OR() 和 BIT_XOR() 操作

对于`BIT_AND()`、`BIT_OR()`和`BIT_XOR()`位函数，结果类型取决于函数参数值是作为二进制字符串还是数字进行评估：

+   当参数值具有二进制字符串类型且参数不是十六进制文字、位文字或`NULL`文字时，会发生二进制字符串评估。否则会发生数值评估，必要时将参数值转换为无符号 64 位整数。

+   二进制字符串评估会产生与参数值相同长度的二进制字符串。如果参数值长度不相等，则会出现`ER_INVALID_BITWISE_OPERANDS_SIZE`错误。如果参数大小超过 511 字节，则会出现`ER_INVALID_BITWISE_AGGREGATE_OPERANDS_SIZE`错误。数值评估会产生一个无符号 64 位整数。

`NULL`值不会影响结果，除非所有值都是`NULL`。在这种情况下，结果是一个中性值，其长度与参数值的长度相同（对于`BIT_AND()`为所有位为 1，对于`BIT_OR()`为所有位为 0，以及`BIT_XOR()`）。

例子：

```sql
mysql> CREATE TABLE t (group_id INT, a VARBINARY(6));
mysql> INSERT INTO t VALUES (1, NULL);
mysql> INSERT INTO t VALUES (1, NULL);
mysql> INSERT INTO t VALUES (2, NULL);
mysql> INSERT INTO t VALUES (2, X'1234');
mysql> INSERT INTO t VALUES (2, X'FF34');
mysql> SELECT HEX(BIT_AND(a)), HEX(BIT_OR(a)), HEX(BIT_XOR(a))
       FROM t GROUP BY group_id;
+-----------------+----------------+-----------------+
| HEX(BIT_AND(a)) | HEX(BIT_OR(a)) | HEX(BIT_XOR(a)) |
+-----------------+----------------+-----------------+
| FFFFFFFFFFFF    | 000000000000   | 000000000000    |
| 1234            | FF34           | ED00            |
+-----------------+----------------+-----------------+
```

### 十六进制文字、位文字和 NULL 文字的特殊处理

为了向后兼容，当所有位参数为十六进制文字、位文字或`NULL`文字时，MySQL 8.0 会在数值上评估位操作。也就是说，如果所有位参数都是未修饰的十六进制文字、位文字或`NULL`文字，则对二进制字符串位参数的位操作不会使用二进制字符串评估。（如果它们是用`_binary`引导符、`BINARY`运算符或其他明确指定为二进制字符串的方式写入的，则不适用于这些文字。）

刚才描述的文字处理与 MySQL 8.0 之前的版本相同。例如：

+   这些位操作在数值上评估文字并产生一个`BIGINT`结果：

    ```sql
    b'0001' | b'0010'
    X'0008' << 8
    ```

+   这些位操作在数值上评估`NULL`并产生一个具有`NULL`值的`BIGINT`结果：

    ```sql
    NULL & NULL
    NULL >> 4
    ```

在 MySQL 8.0 中，您可以通过明确指示至少一个参数是二进制字符串来导致这些操作在二进制字符串上下文中评估参数：

```sql
_binary b'0001' | b'0010'
_binary X'0008' << 8
BINARY NULL & NULL
BINARY NULL >> 4
```

最后两个表达式的结果是`NULL`，就像没有`BINARY`运算符一样，但结果的数据类型是二进制字符串类型而不是整数类型。

### 与 MySQL 5.7 不兼容的位操作

因为 MySQL 8.0 可以原生处理二进制字符串参数的位操作，一些表达式在 MySQL 8.0 中产生的结果与 5.7 中不同。需要注意的五种问题表达式类型是：

```sql
*nonliteral_binary* { & | ^ } *binary*
*binary*  { & | ^ } *nonliteral_binary*
*nonliteral_binary* { << >> } *anything*
~ *nonliteral_binary*
*AGGR_BIT_FUNC*(*nonliteral_binary*)
```

这些表达式在 MySQL 5.7 中返回`BIGINT`，在 8.0 中返回二进制字符串。

符号说明：

+   `{ *`op1`* *`op2`* ... }`：适用于给定表达式类型的运算符列表。

+   *`binary`*：任何类型的二进制字符串参数，包括十六进制文字、位文字或`NULL`文字。

+   *`nonliteral_binary`*：一个不是十六进制文字、位文字或`NULL`文字的二进制字符串值的参数。

+   *`AGGR_BIT_FUNC`*：一个接受位值参数的聚合函数：`BIT_AND()`，`BIT_OR()`，`BIT_XOR()`。

有关如何在 MySQL 5.7 中准备可能的 MySQL 5.7 和 8.0 之间不兼容性的信息，请参阅位函数和运算符，在 MySQL 5.7 参考手册中。
