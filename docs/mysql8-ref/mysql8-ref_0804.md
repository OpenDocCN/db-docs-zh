# 14.8.2 正则表达式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/regexp.html`](https://dev.mysql.com/doc/refman/8.0/en/regexp.html)

**表 14.14 正则表达式函数和运算符**

| 名称 | 描述 |
| --- | --- |
| `NOT REGEXP` | REGEXP 的否定 |
| `REGEXP` | 字符串是否匹配正则表达式 |
| `REGEXP_INSTR()` | 返回匹配正则表达式的子字符串的起始索引 |
| `REGEXP_LIKE()` | 字符串是否匹配正则表达式 |
| `REGEXP_REPLACE()` | 替换匹配正则表达式的子字符串 |
| `REGEXP_SUBSTR()` | 返回匹配正则表达式的子字符串 |
| `RLIKE` | 字符串是否匹配正则表达式 |

正则表达式是指定复杂搜索模式的强大方式。本节讨论了用于正则表达式匹配的函数和运算符，并举例说明了一些特殊字符和结构，可用于正则表达式操作。另请参见第 5.3.4.7 节，“模式匹配”。

MySQL 使用国际化组件 Unicode（ICU）实现正则表达式支持，提供完整的 Unicode 支持并且是多字节安全的。（在 MySQL 8.0.4 之前，MySQL 使用 Henry Spencer 的正则表达式实现，以字节方式操作，不是多字节安全的。有关使用正则表达式的应用程序可能受到实现更改影响的信息，请参见正则表达式兼容性注意事项.）

在 MySQL 8.0.22 之前，可以使用二进制字符串参数与这些函数，但结果不一致。在 MySQL 8.0.22 及更高版本中，使用任何 MySQL 正则表达式函数的二进制字符串将被拒绝，并显示`ER_CHARACTER_SET_MISMATCH`。

+   正则表达式函数和运算符描述

+   正则表达式语法

+   正则表达式资源控制

+   正则表达式兼容性注意事项

#### 正则表达式函数和运算符描述

+   `*`expr`* NOT REGEXP *`pat`*`, `*`expr`* NOT RLIKE *`pat`*`

    这与 `NOT (*`expr`* REGEXP *`pat`*)` 相同。

+   `*`expr`* REGEXP *`pat`*`, `*`expr`* RLIKE *`pat`*`

    如果字符串*`expr`*与模式*`pat`*指定的正则表达式匹配，则返回 1，否则返回 0。如果*`expr`*或*`pat`*为`NULL`，则返回值为`NULL`。

    `REGEXP`和`RLIKE`是`REGEXP_LIKE()`的同义词。

    有关匹配发生的更多信息，请参阅`REGEXP_LIKE()`的描述。

    ```sql
    mysql> SELECT 'Michael!' REGEXP '.*';
    +------------------------+
    | 'Michael!' REGEXP '.*' |
    +------------------------+
    |                      1 |
    +------------------------+
    mysql> SELECT 'new*\n*line' REGEXP 'new\\*.\\*line';
    +---------------------------------------+
    | 'new*\n*line' REGEXP 'new\\*.\\*line' |
    +---------------------------------------+
    |                                     0 |
    +---------------------------------------+
    mysql> SELECT 'a' REGEXP '^[a-d]';
    +---------------------+
    | 'a' REGEXP '^[a-d]' |
    +---------------------+
    |                   1 |
    +---------------------+
    ```

+   [`REGEXP_INSTR(*`expr`*, *`pat`*[, *`pos`*[, *`occurrence`*[, *`return_option`*[, *`match_type`*]]]])`](regexp.html#function_regexp-instr)

    返回字符串*`expr`*中与模式*`pat`*指定的正则表达式匹配的子字符串的起始索引，如果没有匹配则返回 0。如果*`expr`*或*`pat`*为`NULL`，则返回值为`NULL`。字符索引从 1 开始。

    `REGEXP_INSTR()`接受这些可选参数：

    +   *`pos`*：开始搜索的*`expr`*中的位置。如果省略，则默认为 1。

    +   *`occurrence`*：要搜索的匹配的哪个出现。如果省略，则默认为 1。

    +   *`return_option`*：要返回的位置类型。如果此值为 0，`REGEXP_INSTR()`将返回匹配子字符串的第一个字符的位置。如果此值为 1，`REGEXP_INSTR()`将返回匹配子字符串后面的位置。如果省略，则默认为 0。

    +   *`match_type`*：指定如何执行匹配的字符串。其含义如`REGEXP_LIKE()`中描述的。

    有关匹配发生的更多信息，请参阅`REGEXP_LIKE()`的描述。

    ```sql
    mysql> SELECT REGEXP_INSTR('dog cat dog', 'dog');
    +------------------------------------+
    | REGEXP_INSTR('dog cat dog', 'dog') |
    +------------------------------------+
    |                                  1 |
    +------------------------------------+
    mysql> SELECT REGEXP_INSTR('dog cat dog', 'dog', 2);
    +---------------------------------------+
    | REGEXP_INSTR('dog cat dog', 'dog', 2) |
    +---------------------------------------+
    |                                     9 |
    +---------------------------------------+
    mysql> SELECT REGEXP_INSTR('aa aaa aaaa', 'a{2}');
    +-------------------------------------+
    | REGEXP_INSTR('aa aaa aaaa', 'a{2}') |
    +-------------------------------------+
    |                                   1 |
    +-------------------------------------+
    mysql> SELECT REGEXP_INSTR('aa aaa aaaa', 'a{4}');
    +-------------------------------------+
    | REGEXP_INSTR('aa aaa aaaa', 'a{4}') |
    +-------------------------------------+
    |                                   8 |
    +-------------------------------------+
    ```

+   [`REGEXP_LIKE(*`expr`*, *`pat`*[, *`match_type`*])`](regexp.html#function_regexp-like)

    如果字符串*`expr`*与模式*`pat`*指定的正则表达式匹配，则返回 1，否则返回 0。如果*`expr`*或*`pat`*为`NULL`，则返回值为`NULL`。

    模式可以是扩展正则表达式，其语法在正则表达式语法中讨论。模式不必是一个字面字符串。例如，它可以被指定为一个字符串表达式或表列。

    可选的*`match_type`*参数是一个字符串，可以包含任何或所有以下字符，指定如何执行匹配：

    +   `c`：区分大小写匹配。

    +   `i`：不区分大小写匹配。

    +   `m`：多行模式。识别字符串内的行终止符。默认行为是仅在字符串表达式的开头和结尾匹配行终止符。

    +   `n`：`.`字符匹配行终止符。默认情况下，`.`匹配会在行尾停止。

    +   `u`：仅适用于 Unix 换行符。只有换行符被`.`、`^`和`$`匹配操作符识别为换行符。

    如果在*`match_type`*中指定了指定矛盾选项的字符，则最右边的选项优先。

    默认情况下，正则表达式操作在决定字符类型和执行比较时使用*`expr`*和*`pat`*参数的字符集和排序规则。如果参数具有不同的字符集或排序规则，则会应用强制转换规则，如第 12.8.4 节“表达式中的排序规则可转换性”中所述。可以使用显式排序标识符指定参数以更改比较行为。

    ```sql
    mysql> SELECT REGEXP_LIKE('CamelCase', 'CAMELCASE');
    +---------------------------------------+
    | REGEXP_LIKE('CamelCase', 'CAMELCASE') |
    +---------------------------------------+
    |                                     1 |
    +---------------------------------------+
    mysql> SELECT REGEXP_LIKE('CamelCase', 'CAMELCASE' COLLATE utf8mb4_0900_as_cs);
    +------------------------------------------------------------------+
    | REGEXP_LIKE('CamelCase', 'CAMELCASE' COLLATE utf8mb4_0900_as_cs) |
    +------------------------------------------------------------------+
    |                                                                0 |
    +------------------------------------------------------------------+
    ```

    *`match_type`*可以用`c`或`i`字符指定以覆盖默认的大小写敏感性。例外：如果任一参数是二进制字符串，则参数将作为二进制字符串进行大小写敏感处理，即使*`match_type`*包含`i`字符。

    注意

    MySQL 在字符串中使用 C 转义语法（例如，`\n`表示换行符）。如果您希望您的*`expr`*或*`pat`*参数包含文字`\`，则必须将其加倍。（除非启用了`NO_BACKSLASH_ESCAPES` SQL 模式，在这种情况下不使用转义字符。）

    ```sql
    mysql> SELECT REGEXP_LIKE('Michael!', '.*');
    +-------------------------------+
    | REGEXP_LIKE('Michael!', '.*') |
    +-------------------------------+
    |                             1 |
    +-------------------------------+
    mysql> SELECT REGEXP_LIKE('new*\n*line', 'new\\*.\\*line');
    +----------------------------------------------+
    | REGEXP_LIKE('new*\n*line', 'new\\*.\\*line') |
    +----------------------------------------------+
    |                                            0 |
    +----------------------------------------------+
    mysql> SELECT REGEXP_LIKE('a', '^[a-d]');
    +----------------------------+
    | REGEXP_LIKE('a', '^[a-d]') |
    +----------------------------+
    |                          1 |
    +----------------------------+
    ```

    ```sql
    mysql> SELECT REGEXP_LIKE('abc', 'ABC');
    +---------------------------+
    | REGEXP_LIKE('abc', 'ABC') |
    +---------------------------+
    |                         1 |
    +---------------------------+
    mysql> SELECT REGEXP_LIKE('abc', 'ABC', 'c');
    +--------------------------------+
    | REGEXP_LIKE('abc', 'ABC', 'c') |
    +--------------------------------+
    |                              0 |
    +--------------------------------+
    ```

+   [`REGEXP_REPLACE(*`expr`*, *`pat`*, *`repl`*[, *`pos`*[, *`occurrence`*[, *`match_type`*]]])`](regexp.html#function_regexp-replace)

    用替换字符串*`repl`*替换与模式*`pat`*指定的正则表达式匹配的字符串*`expr`*中的出现，并返回结果字符串。如果*`expr`*、*`pat`*或*`repl`*为`NULL`，则返回值为`NULL`。

    `REGEXP_REPLACE()`接受这些可选参数：

    +   *`pos`*：在*`expr`*中开始搜索的位置。如果省略，默认为 1。

    +   *`occurrence`*：要替换的匹配的哪个出现。如果省略，默认为 0（表示“替换所有出现”）。

    +   *`match_type`*：一个指定如何执行匹配的字符串。其含义如`REGEXP_LIKE()`中所述。

    在 MySQL 8.0.17 之前，此函数返回的结果使用`UTF-16`字符集；在 MySQL 8.0.17 及更高版本中，使用搜索匹配的表达式的字符集和排序规则。（Bug #94203，Bug #29308212）

    有关匹配发生的更多信息，请参阅`REGEXP_LIKE()`的描述。

    ```sql
    mysql> SELECT REGEXP_REPLACE('a b c', 'b', 'X');
    +-----------------------------------+
    | REGEXP_REPLACE('a b c', 'b', 'X') |
    +-----------------------------------+
    | a X c                             |
    +-----------------------------------+
    mysql> SELECT REGEXP_REPLACE('abc def ghi', '[a-z]+', 'X', 1, 3);
    +----------------------------------------------------+
    | REGEXP_REPLACE('abc def ghi', '[a-z]+', 'X', 1, 3) |
    +----------------------------------------------------+
    | abc def X                                          |
    +----------------------------------------------------+
    ```

+   [`REGEXP_SUBSTR(*`expr`*, *`pat`*[, *`pos`*[, *`occurrence`*[, *`match_type`*]]])`](regexp.html#function_regexp-substr)

    返回与模式*`pat`*指定的正则表达式匹配的字符串*`expr`*的子字符串，如果没有匹配则返回`NULL`。如果*`expr`*或*`pat`*为`NULL`，则返回值为`NULL`。

    `REGEXP_SUBSTR()`接受这些可选参数：

    +   *`pos`*：在*`expr`*中开始搜索的位置。如果省略，默认为 1。

    +   *`occurrence`*：要搜索的匹配的哪个出现。如果省略，默认为 1。

    +   *`match_type`*：指定如何执行匹配的字符串。其含义与`REGEXP_LIKE()`中描述的相同。

    在 MySQL 8.0.17 之前，此函数返回的结果使用`UTF-16`字符集；在 MySQL 8.0.17 及更高版本中，使用搜索匹配的表达式的字符集和校对规则。（Bug #94203，Bug #29308212）

    有关匹配发生的更多信息，请参阅`REGEXP_LIKE()`的描述。

    ```sql
    mysql> SELECT REGEXP_SUBSTR('abc def ghi', '[a-z]+');
    +----------------------------------------+
    | REGEXP_SUBSTR('abc def ghi', '[a-z]+') |
    +----------------------------------------+
    | abc                                    |
    +----------------------------------------+
    mysql> SELECT REGEXP_SUBSTR('abc def ghi', '[a-z]+', 1, 3);
    +----------------------------------------------+
    | REGEXP_SUBSTR('abc def ghi', '[a-z]+', 1, 3) |
    +----------------------------------------------+
    | ghi                                          |
    +----------------------------------------------+
    ```

#### 正则表达式语法

正则表达式描述了一组字符串。最简单的正则表达式是没有特殊字符的正则表达式。例如，正则表达式`hello`匹配`hello`，不匹配其他任何内容。

复杂的正则表达式使用特定的特殊构造，以便能够匹配多个字符串。例如，正则表达式`hello|world`包含`|`交替运算符，匹配`hello`或`world`。

作为一个更复杂的例子，正则表达式`B[an]*s`匹配任何字符串`Bananas`，`Baaaaas`，`Bs`，以及任何以`B`开头，以`s`结尾，并且在中间包含任意数量的`a`或`n`字符的字符串。

以下列表涵盖了一些基本的特殊字符和构造，可用于正则表达式中。有关 ICU 库支持的完整正则表达式语法的信息，请访问[国际 Unicode 组件网站](https://unicode-org.github.io/icu/userguide/)。

+   `^`

    匹配字符串的开头。

    ```sql
    mysql> SELECT REGEXP_LIKE('fo\nfo', '^fo$');                   -> 0
    mysql> SELECT REGEXP_LIKE('fofo', '^fo');                      -> 1
    ```

+   `$`

    匹配字符串的结尾。

    ```sql
    mysql> SELECT REGEXP_LIKE('fo\no', '^fo\no$');                 -> 1
    mysql> SELECT REGEXP_LIKE('fo\no', '^fo$');                    -> 0
    ```

+   `.`

    匹配任何字符（包括回车和换行符，尽管要匹配字符串中间的这些字符，必须给定`m`（多行）匹配控制字符或`(?m)`模式内修改器）。

    ```sql
    mysql> SELECT REGEXP_LIKE('fofo', '^f.*$');                    -> 1
    mysql> SELECT REGEXP_LIKE('fo\r\nfo', '^f.*$');                -> 0
    mysql> SELECT REGEXP_LIKE('fo\r\nfo', '^f.*$', 'm');           -> 1
    mysql> SELECT REGEXP_LIKE('fo\r\nfo', '(?m)^f.*$');           -> 1
    ```

+   `a*`

    匹配零个或多个`a`字符的序列。

    ```sql
    mysql> SELECT REGEXP_LIKE('Ban', '^Ba*n');                     -> 1
    mysql> SELECT REGEXP_LIKE('Baaan', '^Ba*n');                   -> 1
    mysql> SELECT REGEXP_LIKE('Bn', '^Ba*n');                      -> 1
    ```

+   `a+`

    匹配一个或多个`a`字符的序列。

    ```sql
    mysql> SELECT REGEXP_LIKE('Ban', '^Ba+n');                     -> 1
    mysql> SELECT REGEXP_LIKE('Bn', '^Ba+n');                      -> 0
    ```

+   `a?`

    匹配零个或一个`a`字符。

    ```sql
    mysql> SELECT REGEXP_LIKE('Bn', '^Ba?n');                      -> 1
    mysql> SELECT REGEXP_LIKE('Ban', '^Ba?n');                     -> 1
    mysql> SELECT REGEXP_LIKE('Baan', '^Ba?n');                    -> 0
    ```

+   `de|abc`

    交替；匹配`de`或`abc`中的任一序列。

    ```sql
    mysql> SELECT REGEXP_LIKE('pi', 'pi|apa');                     -> 1
    mysql> SELECT REGEXP_LIKE('axe', 'pi|apa');                    -> 0
    mysql> SELECT REGEXP_LIKE('apa', 'pi|apa');                    -> 1
    mysql> SELECT REGEXP_LIKE('apa', '^(pi|apa)$');                -> 1
    mysql> SELECT REGEXP_LIKE('pi', '^(pi|apa)$');                 -> 1
    mysql> SELECT REGEXP_LIKE('pix', '^(pi|apa)$');                -> 0
    ```

+   `(abc)*`

    匹配序列`abc`的零个或多个实例。

    ```sql
    mysql> SELECT REGEXP_LIKE('pi', '^(pi)*$');                    -> 1
    mysql> SELECT REGEXP_LIKE('pip', '^(pi)*$');                   -> 0
    mysql> SELECT REGEXP_LIKE('pipi', '^(pi)*$');                  -> 1
    ```

+   `{1}`, `{2,3}`

    重复；`{*`n`*}`和`{*`m`*,*`n`*}`表示一种更通用的写正则表达式的方式，可以匹配前一个原子（或“片段”）的模式的多个出现。*`m`*和*`n`*是整数。

    +   `a*`

        可以写成`a{0,}`。

    +   `a+`

        可以写成`a{1,}`。

    +   `a?`

        可以写成`a{0,1}`。

    要更精确地说，`a{*`n`*}` 精确匹配 *`n`* 个 `a`。`a{*`n`*,}` 匹配 *`n`* 个或更多个 `a`。`a{*`m`*,*`n`*}` 匹配 *`m`* 到 *`n`* 个 `a`，包括两端。如果同时给出 *`m`* 和 *`n`*，则 *`m`* 必须小于或等于 *`n`*。

    ```sql
    mysql> SELECT REGEXP_LIKE('abcde', 'a[bcd]{2}e');              -> 0
    mysql> SELECT REGEXP_LIKE('abcde', 'a[bcd]{3}e');              -> 1
    mysql> SELECT REGEXP_LIKE('abcde', 'a[bcd]{1,10}e');           -> 1
    ```

+   `[a-dX]`, `[^a-dX]`

    匹配任何是（或不是，如果使用 `^`）`a`、`b`、`c`、`d` 或 `X` 的字符。两个字符之间的 `-` 形成一个范围，匹配从第一个字符到第二个字符的所有字符。例如，`[0-9]` 匹配任何十进制数字。要包含文字 `]` 字符，它必须紧跟在开方括号 `[` 之后。要包含文字 `-` 字符，它必须写在最前面或最后面。在 `[]` 对中没有定义特殊含义的任何字符只匹配它本身。

    ```sql
    mysql> SELECT REGEXP_LIKE('aXbc', '[a-dXYZ]');                 -> 1
    mysql> SELECT REGEXP_LIKE('aXbc', '^[a-dXYZ]$');               -> 0
    mysql> SELECT REGEXP_LIKE('aXbc', '^[a-dXYZ]+$');              -> 1
    mysql> SELECT REGEXP_LIKE('aXbc', '^[^a-dXYZ]+$');             -> 0
    mysql> SELECT REGEXP_LIKE('gheis', '^[^a-dXYZ]+$');            -> 1
    mysql> SELECT REGEXP_LIKE('gheisa', '^[^a-dXYZ]+$');           -> 0
    ```

+   `[=character_class=]`

    在方括号表达式（使用 `[` 和 `]` 编写）中，`[=character_class=]` 表示一个等价类。它匹配所有具有相同排序值的字符，包括它本身。例如，如果 `o` 和 `(+)` 是等价类的成员，则 `[[=o=]]`、`[[=(+)=]]` 和 `[o(+)]` 都是同义词。等价类不能用作范围的端点。

+   `[:character_class:]`

    在方括号表达式（使用 `[` 和 `]` 编写）中，`[:character_class:]` 表示一个匹配属于该类的所有字符的字符类。以下表列出了标准类名。这些名称代表 `ctype(3)` 手册页中定义的字符类。特定区域设置可能提供其他类名。字符类不能用作范围的端点。

    | 字符类名 | 含义 |
    | --- | --- |
    | `alnum` | 字母数字字符 |
    | `alpha` | 字母字符 |
    | `blank` | 空白字符 |
    | `cntrl` | 控制字符 |
    | `digit` | 数字字符 |
    | `graph` | 图形字符 |
    | `lower` | 小写字母字符 |
    | `print` | 图形或空格字符 |
    | `punct` | 标点字符 |
    | `space` | 空格、制表符、换行符和回车符 |
    | `upper` | 大写字母字符 |
    | `xdigit` | 十六进制数字字符 |
    | 字符类名 | 含义 |

    ```sql
    mysql> SELECT REGEXP_LIKE('justalnums', '[[:alnum:]]+');       -> 1
    mysql> SELECT REGEXP_LIKE('!!', '[[:alnum:]]+');               -> 0
    ```

    由于 ICU 知道 `utf16_general_ci` 中的所有字母字符，因此某些字符类的性能可能不如字符范围高。例如，`[a-zA-Z]` 比 `[[:alpha:]]` 更快，`[0-9]` 通常比 `[[:digit:]]` 更快。如果您正在迁移使用 `[[:alpha:]]` 或 `[[:digit:]]` 的应用程序从旧版本的 MySQL，您应该将这些替换为等效的范围以在 MySQL 8.0 中使用。

要在正则表达式中使用特殊字符的文字实例，请在其前面加上两个反斜杠（\）字符。MySQL 解析器解释其中一个反斜杠，而正则表达式库解释另一个。例如，要匹配包含特殊`+`字符的字符串`1+2`，只有以下正则表达式中的最后一个是正确的：

```sql
mysql> SELECT REGEXP_LIKE('1+2', '1+2');                       -> 0
mysql> SELECT REGEXP_LIKE('1+2', '1\+2');                      -> 0
mysql> SELECT REGEXP_LIKE('1+2', '1\\+2');                     -> 1
```

#### 正则表达式资源控制

`REGEXP_LIKE()`和类似函数使用可以通过设置系统变量来控制的资源：

+   匹配引擎使用内部堆栈的内存。要控制堆栈的最大可用内存（以字节为单位），请设置`regexp_stack_limit`系统变量。

+   匹配引擎以步骤方式运行。要控制引擎执行的最大步骤数（从而间接影响执行时间），请设置`regexp_time_limit`系统变量。因为此限制表示为步骤数，所以它只间接影响执行时间。通常，它的数量级为毫秒。

#### 正则表达式兼容性考虑

在 MySQL 8.0.4 之前，MySQL 使用 Henry Spencer 正则表达式库来支持正则表达式操作，而不是国际 Unicode 组件（ICU）。以下讨论描述了 Spencer 和 ICU 库之间可能影响应用程序的差异：

+   使用 Spencer 库，`REGEXP`和`RLIKE`运算符以字节方式工作，因此它们不是多字节安全的，可能会在使用多字节字符集时产生意外结果。此外，这些运算符通过它们的字节值比较字符，即使给定排序将它们视为相等，重音字符也可能不相等。

    ICU 具有完整的 Unicode 支持，并且是多字节安全的。其正则表达式函数将所有字符串视为`UTF-16`。您应该记住，位置索引是基于 16 位块而不是代码点的。这意味着，当传递给这些函数时，使用多个块的字符可能会产生意想不到的结果，例如下面所示：

    ```sql
    mysql> SELECT REGEXP_INSTR('🍣🍣b', 'b');
    +--------------------------+
    | REGEXP_INSTR('??b', 'b') |
    +--------------------------+
    |                        5 |
    +--------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT REGEXP_INSTR('🍣🍣bxxx', 'b', 4);
    +--------------------------------+
    | REGEXP_INSTR('??bxxx', 'b', 4) |
    +--------------------------------+
    |                              5 |
    +--------------------------------+
    1 row in set (0.00 sec)
    ```

    Unicode 基本多文种平面内的字符（包括大多数现代语言使用的字符）在这方面是安全的：

    ```sql
    mysql> SELECT REGEXP_INSTR('бжb', 'b');
    +----------------------------+
    | REGEXP_INSTR('бжb', 'b')   |
    +----------------------------+
    |                          3 |
    +----------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT REGEXP_INSTR('עבb', 'b');
    +----------------------------+
    | REGEXP_INSTR('עבb', 'b')   |
    +----------------------------+
    |                          3 |
    +----------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT REGEXP_INSTR('µå周çб', '周');
    +------------------------------------+
    | REGEXP_INSTR('µå周çб', '周')       |
    +------------------------------------+
    |                                  3 |
    +------------------------------------+
    1 row in set (0.00 sec)
    ```

    表情符号，例如第一个两个示例中使用的“寿司”字符`🍣`（U+1F363），不包括在基本多语言平面中，而是在 Unicode 的补充多语言平面中。当`REGEXP_SUBSTR()`或类似函数从字符中间开始搜索时，表情符号和其他 4 字节字符可能会出现问题。以下示例中的两个语句中的每个都从第一个参数的第二个 2 字节位置开始。第一个语句适用于仅由 2 字节（BMP）字符组成的字符串。第二个语句包含 4 字节字符，结果中错误解释了这些字符，因为前两个字节被剥离，因此字符数据的其余部分错位。

    ```sql
    mysql> SELECT REGEXP_SUBSTR('周周周周', '.*', 2);
    +----------------------------------------+
    | REGEXP_SUBSTR('周周周周', '.*', 2)     |
    +----------------------------------------+
    | 周周周                                 |
    +----------------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT REGEXP_SUBSTR('🍣🍣🍣🍣', '.*', 2);
    +--------------------------------+
    | REGEXP_SUBSTR('????', '.*', 2) |
    +--------------------------------+
    | ?㳟揘㳟揘㳟揘                  |
    +--------------------------------+
    1 row in set (0.00 sec)
    ```

+   对于`.`运算符，Spencer 库在字符串表达式中的任何位置（包括中间）匹配行终止字符（回车，换行）。要在 ICU 中匹配字符串中间的行终止字符，请指定`m`匹配控制字符。

+   Spencer 库支持单词开头和单词结尾边界标记（`[[:<:]]`和`[[:>:]]`表示）。ICU 不支持。对于 ICU，您可以使用`\b`来匹配单词边界；双反斜杠因为 MySQL 将其解释为字符串中的转义字符。

+   Spencer 库支持整理元素括号表达式（`[.characters.]`表示）。ICU 不支持。

+   对于重复计数（`{n}`和`{m,n}`表示），Spencer 库最多为 255。ICU 没有此限制，尽管可以通过设置`regexp_time_limit`系统变量来限制匹配引擎步骤的最大数量。

+   ICU 将括号解释为元字符。要在正则表达式中指定字面上的开放`(`或关闭括号`)，必须进行转义：

    ```sql
    mysql> SELECT REGEXP_LIKE('(', '(');
    ERROR 3692 (HY000): Mismatched parenthesis in regular expression.
    mysql> SELECT REGEXP_LIKE('(', '\\(');
    +-------------------------+
    | REGEXP_LIKE('(', '\\(') |
    +-------------------------+
    |                       1 |
    +-------------------------+
    mysql> SELECT REGEXP_LIKE(')', ')');
    ERROR 3692 (HY000): Mismatched parenthesis in regular expression.
    mysql> SELECT REGEXP_LIKE(')', '\\)');
    +-------------------------+
    | REGEXP_LIKE(')', '\\)') |
    +-------------------------+
    |                       1 |
    +-------------------------+
    ```

+   ICU 还将方括号解释为元字符，但只有开放方括号需要转义才能用作字面字符：

    ```sql
    mysql> SELECT REGEXP_LIKE('[', '[');
    ERROR 3696 (HY000): The regular expression contains an
    unclosed bracket expression.
    mysql> SELECT REGEXP_LIKE('[', '\\[');
    +-------------------------+
    | REGEXP_LIKE('[', '\\[') |
    +-------------------------+
    |                       1 |
    +-------------------------+
    mysql> SELECT REGEXP_LIKE(']', ']');
    +-----------------------+
    | REGEXP_LIKE(']', ']') |
    +-----------------------+
    |                     1 |
    +-----------------------+
    ```
