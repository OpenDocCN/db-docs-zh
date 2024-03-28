# 12.3.8 字符集引导符

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-introducer.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-introducer.html)

字符串文字、十六进制文字或位值文字可以有可选的字符集引导符和`COLLATE`子句，以指定它为使用特定字符集和校对规则的字符串：

```sql
[_*charset_name*] *literal* [COLLATE *collation_name*]
```

`_*`charset_name`*`表达式在形式上称为*引导符*。它告诉解析器，“接下来的字符串使用字符集*`charset_name`*。” 引导符不会像`CONVERT()`那样将字符串更改为引导符字符集。它不会更改字符串值，尽管可能会发生填充。引导符只是一个信号。

对于字符串文字，引导符和字符串之间的空格是允许但是可选的。

对于字符集文字，引导符表示后续字符串的字符集，但不会改变解析器如何在字符串内执行转义处理。转义始终由解析器根据`character_set_connection`给定的字符集进行解释。有关更多讨论和示例，请参见第 12.3.6 节，“字符串文字的字符集和校对规则”。

例子：

```sql
SELECT 'abc';
SELECT _latin1'abc';
SELECT _binary'abc';
SELECT _utf8mb4'abc' COLLATE utf8mb4_danish_ci;

SELECT _latin1 X'4D7953514C';
SELECT _utf8mb4 0x4D7953514C COLLATE utf8mb4_danish_ci;

SELECT _latin1 b'1000001';
SELECT _utf8mb4 0b1000001 COLLATE utf8mb4_danish_ci;
```

字符集引导符和`COLLATE`子句按照标准 SQL 规范实现。

字符串文字可以通过使用`_binary`引导符指定为二进制字符串。十六进制文字和位值文字默认为二进制字符串，因此允许使用`_binary`，但通常是不必要的。在 MySQL 8.0 及更高版本中，位操作允许数字或二进制字符串参数，但默认情况下将十六进制和位文字视为数字。为了明确指定这些文字的二进制字符串上下文，对于这些文字至少一个参数使用`_binary`引导符：

```sql
mysql> SET @v1 = X'000D' | X'0BC0';
mysql> SET @v2 = _binary X'000D' | X'0BC0';
mysql> SELECT HEX(@v1), HEX(@v2);
+----------+----------+
| HEX(@v1) | HEX(@v2) |
+----------+----------+
| BCD      | 0BCD     |
+----------+----------+
```

对于位操作，显示的结果看起来相似，但没有`_binary`的结果是一个`BIGINT`值，而有`_binary`的结果是一个二进制字符串。由于结果类型的差异，显示的值也不同：高位 0 数字不会显示在数值结果中。

MySQL 确定字符串文字、十六进制文字或位值文字的字符集和校对规则的方式如下：

+   如果同时指定了*`_charset_name`*和`COLLATE *`collation_name`*`，则使用字符集*`charset_name`*和校对规则*`collation_name`*。*`collation_name`*必须是*`charset_name`*的允许校对规则。

+   如果指定了*`_charset_name`*但未指定`COLLATE`，则使用字符集*`charset_name`*及其默认排序规则。要查看每个字符集的默认排序规则，请使用`SHOW CHARACTER SET`语句或查询`INFORMATION_SCHEMA` `CHARACTER_SETS`表。

+   如果未指定*`_charset_name`*但指定了`COLLATE *`collation_name`*`：

    +   对于字符字符串字面值，使用由`character_set_connection`系统变量给出的连接默认字符集和排序规则*`collation_name`*。*`collation_name`*必须是连接默认字符集的允许排序规则。

    +   对于十六进制字面值或位值字面值，唯一允许的排序规则是`binary`，因为这些类型的字面值默认为二进制字符串。

+   否则（既未指定*`_charset_name`*也未指定`COLLATE *`collation_name`*`）：

    +   对于字符字符串字面值，使用由`character_set_connection`和`collation_connection`系统变量给出的连接默认字符集和排序规则。

    +   对于十六进制字面值或位值字面值，字符集和排序规则为`binary`。

例子：

+   具有`latin1`字符集和`latin1_german1_ci`排序规则的非二进制字符串：

    ```sql
    SELECT _latin1'Müller' COLLATE latin1_german1_ci;
    SELECT _latin1 X'0A0D' COLLATE latin1_german1_ci;
    SELECT _latin1 b'0110' COLLATE latin1_german1_ci;
    ```

+   具有`utf8mb4`字符集及其默认排序规则（即`utf8mb4_0900_ai_ci`）的非二进制字符串：

    ```sql
    SELECT _utf8mb4'Müller';
    SELECT _utf8mb4 X'0A0D';
    SELECT _utf8mb4 b'0110';
    ```

+   具有`binary`字符集及其默认排序规则（即`binary`）的二进制字符串：

    ```sql
    SELECT _binary'Müller';
    SELECT X'0A0D';
    SELECT b'0110';
    ```

    十六进制字面值和位值字面值不需要引导符，因为它们默认为二进制字符串。

+   一个具有连接默认字符集和`utf8mb4_0900_ai_ci`排序规则的非二进制字符串（如果连接字符集不是`utf8mb4`，则失败）：

    ```sql
    SELECT 'Müller' COLLATE utf8mb4_0900_ai_ci;
    ```

    这个构造（`COLLATE` only）对十六进制字面值或位字面值不起作用，因为它们的字符集是`binary`，无论连接字符集如何，而`binary`与`utf8mb4_0900_ai_ci`排序规则不兼容。在没有引导符的情况下，唯一允许的`COLLATE`子句是`COLLATE binary`。

+   一个具有连接默认字符集和排序规则的字符串：

    ```sql
    SELECT 'Müller';
    ```
