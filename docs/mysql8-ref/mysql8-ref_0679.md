# 12.3.6 字符字符串字面量的字符集和校对规则

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-literal.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-literal.html)

每个字符字符串字面量都有一个字符集和一个校对规则。

对于简单语句`SELECT '*`string`*'`，该字符串具有由`character_set_connection`和`collation_connection`系统变量定义的连接默认字符集和���对规则。

字符字符串字面量可以具有可选的字符集引导符和`COLLATE`子句，以指定其为使用特定字符集和校对规则的字符串：

```sql
[_*charset_name*]'*string*' [COLLATE *collation_name*]
```

`_*`charset_name`*`表达式在形式上称为*引导符*。它告诉解析器，“接下来的字符串使用字符集*`charset_name`*。”引导符不会像`CONVERT()`那样将字符串转换为引导符字符集。它不会改变字符串值，尽管可能会发生填充。引导符只是一个信号。参见第 12.3.8 节，“字符集引导符”。

例子：

```sql
SELECT 'abc';
SELECT _latin1'abc';
SELECT _binary'abc';
SELECT _utf8mb4'abc' COLLATE utf8mb4_danish_ci;
```

字符集引导符和`COLLATE`子句按照标准 SQL 规范实现。

MySQL 确定字符字符串字面量的字符集和校对规则的方式如下：

+   如果同时指定了*`_charset_name`*和`COLLATE *`collation_name`*`，则使用字符集*`charset_name`*和校对规则*`collation_name`*。*`collation_name`*必须是*`charset_name`*的允许校对规则。

+   如果*`_charset_name`*被指定但未指定`COLLATE`，则使用字符集*`charset_name`*及其默认校对规则。要查看每个字符集的默认校对规则，请使用`SHOW CHARACTER SET`语句或查询`INFORMATION_SCHEMA` `CHARACTER_SETS`表。

+   如果未指定*`_charset_name`*但指定了`COLLATE *`collation_name`*`，则使用由`character_set_connection`系统变量给出的连接默认字符集和校对规则*`collation_name`*。*`collation_name`*必须是连接默认字符集的允许校对规则。

+   否则（既未指定*`_charset_name`*也未指定`COLLATE *`collation_name`*`），则使用由`character_set_connection`和`collation_connection`系统变量给出的连接默认字符集和校对规则。

例子：

+   一个使用`latin1`字符集和`latin1_german1_ci`校对规则的非二进制字符串：

    ```sql
    SELECT _latin1'Müller' COLLATE latin1_german1_ci;
    ```

+   一个带有`utf8mb4`字符集和其默认排序规则（即`utf8mb4_0900_ai_ci`）的非二进制字符串:

    ```sql
    SELECT _utf8mb4'Müller';
    ```

+   一个带有`binary`字符集和其默认排序规则（即`binary`）的二进制字符串:

    ```sql
    SELECT _binary'Müller';
    ```

+   一个带有连接默认字符集和`utf8mb4_0900_ai_ci`排序规则的非二进制字符串（如果连接字符集不是`utf8mb4`，则失败）:

    ```sql
    SELECT 'Müller' COLLATE utf8mb4_0900_ai_ci;
    ```

+   一个带有连接默认字符集和排序规则的字符串:

    ```sql
    SELECT 'Müller';
    ```

一个引导符指示了接下来字符串的字符集，但不会改变解析器在字符串内部执行转义处理的方式。转义始终由解析器根据`character_set_connection`给定的字符集来解释。

以下示例显示，即使有引导符存在，转义处理仍然使用`character_set_connection`。这些示例使用`SET NAMES`（更改`character_set_connection`的内容，如第 12.4 节“连接字符集和排序规则”中所讨论的），并使用`HEX()`函数显示结果字符串，以便查看确切的字符串内容。

示例 1:

```sql
mysql> SET NAMES latin1;
mysql> SELECT HEX('à\n'), HEX(_sjis'à\n');
+------------+-----------------+
| HEX('à\n')  | HEX(_sjis'à\n')  |
+------------+-----------------+
| E00A       | E00A            |
+------------+-----------------+
```

这里，`à`（十六进制值`E0`）后面跟着`\n`，这是换行的转义序列。转义序列使用`latin1`的`character_set_connection`值来产生一个字面换行（十六进制值`0A`）。即使对于第二个字符串也是如此。也就是说，`_sjis`引导符不会影响解析器的转义处理。

示例 2:

```sql
mysql> SET NAMES sjis;
mysql> SELECT HEX('à\n'), HEX(_latin1'à\n');
+------------+-------------------+
| HEX('à\n')  | HEX(_latin1'à\n')  |
+------------+-------------------+
| E05C6E     | E05C6E            |
+------------+-------------------+
```

这里，`character_set_connection`是`sjis`，一个字符集，其中`à`后面跟着`\`（十六进制值`05`和`5C`）是一个有效的多字节字符。因此，字符串的前两个字节被解释为一个单个的`sjis`字符，`\`不被解释为转义字符。接下来的`n`（十六进制值`6E`）不被解释为转义序列的一部分。即使对于第二个字符串也是如此；`_latin1`引导符不会影响转义处理。
