> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-masking-plugin-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/data-masking-plugin-functions.html)

#### 8.5.3.4 MySQL 企业数据脱敏和去标识化插件函数描述

MySQL 企业数据脱敏和去标识化插件库包括多个函数，可分为以下类别：

+   数据脱敏插件函数

+   随机数据生成插件函数

+   随机数据基于字典的插件函数

从 MySQL 8.0.19 开始，这些函数支持单字节`latin1`字符集用于字符串参数和返回值。在 MySQL 8.0.19 之前，这些函数将字符串参数视为二进制字符串（这意味着它们不区分大小写），并且字符串返回值为二进制字符串。您可以通过以下方式查看返回值字符集的差异：

MySQL 8.0.19 及更高版本：

```sql
mysql> SELECT CHARSET(gen_rnd_email());
+--------------------------+
| CHARSET(gen_rnd_email()) |
+--------------------------+
| latin1                   |
+--------------------------+
```

MySQL 8.0.19 之前：

```sql
mysql> SELECT CHARSET(gen_rnd_email());
+--------------------------+
| CHARSET(gen_rnd_email()) |
+--------------------------+
| binary                   |
+--------------------------+
```

对于任何版本，如果字符串返回值应该使用不同的字符集，请进行转换。以下示例显示如何将`gen_rnd_email()`的结果转换为`utf8mb4`字符集：

```sql
SET @email = CONVERT(gen_rnd_email() USING utf8mb4);
```

要明确生成二进制字符串（例如，生成 MySQL 8.0.19 之前版本的结果），请执行以下操作：

```sql
SET @email = CONVERT(gen_rnd_email() USING binary);
```

有时可能需要转换字符串参数，如使用脱敏数据进行客户识别中所示。

如果从**mysql**客户端内调用 MySQL 企业数据脱敏和去标识化函数，则二进制字符串结果将以十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参见第 6.5.1 节，“mysql — MySQL 命令行客户端”。

##### 数据脱敏插件函数

本节中的每个插件函数都对其字符串参数执行脱敏操作并返回脱敏结果。

+   [`mask_inner(*`str`*, *`margin1`*, *`margin2`* [, *`mask_char`*])`](data-masking-plugin-functions.html#function_mask-inner-plugin)

    脱敏字符串的内部部分，保留末尾不变，并返回结果。可以指定可选的脱敏字符。

    参数：

    +   *`str`*: 需要脱敏的字符串。

    +   *`margin1`*：非负整数，指定要保留未掩码的字符串左端字符数。如果值为 0，则不保留左端字符未掩码。

    +   *`margin2`*：非负整数，指定要保留未掩码的字符串右端字符数。如果值为 0，则不保留右端字符未掩码。

    +   *`mask_char`*：（可选）用于掩码的单个字符。如果未提供*`mask_char`*，默认为`'X'`。

        掩码字符必须是单字节字符。尝试使用多字节字符会产生错误。

    返回值：

    掩码字符串，如果任一边距为负则为`NULL`。

    如果边距值的总和大于参数长度，则不进行掩码，参数返回不变。

    示例：

    ```sql
    mysql> SELECT mask_inner('abcdef', 1, 2), mask_inner('abcdef',0, 5);
    +----------------------------+---------------------------+
    | mask_inner('abcdef', 1, 2) | mask_inner('abcdef',0, 5) |
    +----------------------------+---------------------------+
    | aXXXef                     | Xbcdef                    |
    +----------------------------+---------------------------+
    mysql> SELECT mask_inner('abcdef', 1, 2, '*'), mask_inner('abcdef',0, 5, '#');
    +---------------------------------+--------------------------------+
    | mask_inner('abcdef', 1, 2, '*') | mask_inner('abcdef',0, 5, '#') |
    +---------------------------------+--------------------------------+
    | a***ef                          | #bcdef                         |
    +---------------------------------+--------------------------------+
    ```

+   [`mask_outer(*`str`*, *`margin1`*, *`margin2`* [, *`mask_char`*])`](data-masking-plugin-functions.html#function_mask-outer-plugin)

    掩盖字符串的左右端，保留内部未掩码，并返回结果。可以指定一个可选的掩码字符。

    参数：

    +   *`str`*：要掩码的字符串。

    +   *`margin1`*：非负整数，指定要掩码的字符串左端字符数。如果值为 0，则不掩盖左端字符。

    +   *`margin2`*：非负整数，指定要掩码的字符串右端字符数。如果值为 0，则不掩盖右端字符。

    +   *`mask_char`*：（可选）用于掩码的单个字符。如果未提供*`mask_char`*，默认为`'X'`。

        掩码字符必须是单字节字符。尝试使用多字节字符会产生错误。

    返回值：

    掩码字符串，如果任一边距为负则为`NULL`。

    如果边距值的总和大于参数长度，则整个参数将被掩码。

    示例：

    ```sql
    mysql> SELECT mask_outer('abcdef', 1, 2), mask_outer('abcdef',0, 5);
    +----------------------------+---------------------------+
    | mask_outer('abcdef', 1, 2) | mask_outer('abcdef',0, 5) |
    +----------------------------+---------------------------+
    | XbcdXX                     | aXXXXX                    |
    +----------------------------+---------------------------+
    mysql> SELECT mask_outer('abcdef', 1, 2, '*'), mask_outer('abcdef',0, 5, '#');
    +---------------------------------+--------------------------------+
    | mask_outer('abcdef', 1, 2, '*') | mask_outer('abcdef',0, 5, '#') |
    +---------------------------------+--------------------------------+
    | *bcd**                          | a#####                         |
    +---------------------------------+--------------------------------+
    ```

+   `mask_pan(*`str`*)`

    掩码支付卡主帐号并返回除了最后四位以外的所有数字都替换为`'X'`字符的数字。

    参数：

    +   *`str`*：要掩码的字符串。字符串必须适合主帐号的长度，但不进行其他检查。

    返回值：

    掩盖的支付号码作为字符串。如果参数长度不足，则返回不变。

    示例：

    ```sql
    mysql> SELECT mask_pan(gen_rnd_pan());
    +-------------------------+
    | mask_pan(gen_rnd_pan()) |
    +-------------------------+
    | XXXXXXXXXXXX9102        |
    +-------------------------+
    mysql> SELECT mask_pan(gen_rnd_pan(19));
    +---------------------------+
    | mask_pan(gen_rnd_pan(19)) |
    +---------------------------+
    | XXXXXXXXXXXXXXX8268       |
    +---------------------------+
    mysql> SELECT mask_pan('a*Z');
    +-----------------+
    | mask_pan('a*Z') |
    +-----------------+
    | a*Z             |
    +-----------------+
    ```

+   `mask_pan_relaxed(*`str`*)`

    掩码支付卡主帐号并返回除了前六位和最后四位以外的所有数字都替换为`'X'`字符的数字。前六位数字表示支付卡发行者。

    参数：

    +   *`str`*：要掩码的字符串。字符串必须适合主帐号的长度，但不进行其他检查。

    返回值：

    掩盖的支付号码作为字符串。如果参数长度不足，则返回不变。

    示例：

    ```sql
    mysql> SELECT mask_pan_relaxed(gen_rnd_pan());
    +---------------------------------+
    | mask_pan_relaxed(gen_rnd_pan()) |
    +---------------------------------+
    | 551279XXXXXX3108                |
    +---------------------------------+
    mysql> SELECT mask_pan_relaxed(gen_rnd_pan(19));
    +-----------------------------------+
    | mask_pan_relaxed(gen_rnd_pan(19)) |
    +-----------------------------------+
    | 462634XXXXXXXXX6739               |
    +-----------------------------------+
    mysql> SELECT mask_pan_relaxed('a*Z');
    +-------------------------+
    | mask_pan_relaxed('a*Z') |
    +-------------------------+
    | a*Z                     |
    +-------------------------+
    ```

+   `mask_ssn(*`str`*)`

    掩码美国社会保障号码，并将除最后四位数字外的所有内容替换为`'X'`字符。

    参数：

    +   *`str`*：要掩码的字符串。字符串必须是 11 个字符长。

    返回值：

    掩码的社会保障号码作为字符串，如果参数长度不正确则返回错误。

    示例：

    ```sql
    mysql> SELECT mask_ssn('909-63-6922'), mask_ssn('abcdefghijk');
    +-------------------------+-------------------------+
    | mask_ssn('909-63-6922') | mask_ssn('abcdefghijk') |
    +-------------------------+-------------------------+
    | XXX-XX-6922             | XXX-XX-hijk             |
    +-------------------------+-------------------------+
    mysql> SELECT mask_ssn('909');
    ERROR 1123 (HY000): Can't initialize function 'mask_ssn'; MASK_SSN: Error:
    String argument width too small
    mysql> SELECT mask_ssn('123456789123456789');
    ERROR 1123 (HY000): Can't initialize function 'mask_ssn'; MASK_SSN: Error:
    String argument width too large
    ```

##### 随机数据生成插件函数

本节中的插件函数会为不同类型的数据生成随机值。在可能的情况下，生成的值具有用于演示或测试值的特征，以避免将它们误认为是合法数据。例如，`gen_rnd_us_phone()` 返回一个使用未分配给实际使用电话号码的 555 区号的美国电话号码。各个函数描述会说明此原则的任何例外情况。

+   `gen_range(*`lower`*, *`upper`*)`

    生成从指定范围中选择的随机数。

    参数：

    +   *`lower`*：指定范围下限的整数。

    +   *`upper`*：指定范围上限的整数，必须不小于下限。

    返回值：

    一个在从*`lower`*到*`upper`*（包括）范围内的随机整数，如果*`upper`*参数小于*`lower`*则返回`NULL`。

    示例：

    ```sql
    mysql> SELECT gen_range(100, 200), gen_range(-1000, -800);
    +---------------------+------------------------+
    | gen_range(100, 200) | gen_range(-1000, -800) |
    +---------------------+------------------------+
    |                 177 |                   -917 |
    +---------------------+------------------------+
    mysql> SELECT gen_range(1, 0);
    +-----------------+
    | gen_range(1, 0) |
    +-----------------+
    |            NULL |
    +-----------------+
    ```

+   `gen_rnd_email()`

    生成`example.com`域中的随机电子邮件地址。

    参数：

    无。

    返回值：

    作为字符串的随机电子邮件地址。

    示例：

    ```sql
    mysql> SELECT gen_rnd_email();
    +---------------------------+
    | gen_rnd_email()           |
    +---------------------------+
    | ijocv.mwvhhuf@example.com |
    +---------------------------+
    ```

+   [`gen_rnd_pan([*`size`*])`](data-masking-plugin-functions.html#function_gen-rnd-pan-plugin)

    生成随机支付卡主帐号。该号码通过 Luhn 检查（执行校验和验证以对抗检查位的算法）。

    警告

    从`gen_rnd_pan()`返回的值仅应用于测试目的，不适合发布。无法保证给定的返回值未分配给合法支付账户。如果需要发布`gen_rnd_pan()`结果，请考虑使用`mask_pan()`或`mask_pan_relaxed()`进行掩码处理。

    参数：

    +   *`size`*：（可选）指定结果大小的整数。如果未给出*`size`*，则默认为 16。如果给出，*`size`*必须是 12 到 19 范围内的整数。

    返回值：

    作为字符串的随机支付号码，如果给定超出允许范围的*`size`*参数，则返回`NULL`。

    示例：

    ```sql
    mysql> SELECT mask_pan(gen_rnd_pan());
    +-------------------------+
    | mask_pan(gen_rnd_pan()) |
    +-------------------------+
    | XXXXXXXXXXXX5805        |
    +-------------------------+
    mysql> SELECT mask_pan(gen_rnd_pan(19));
    +---------------------------+
    | mask_pan(gen_rnd_pan(19)) |
    +---------------------------+
    | XXXXXXXXXXXXXXX5067       |
    +---------------------------+
    mysql> SELECT mask_pan_relaxed(gen_rnd_pan());
    +---------------------------------+
    | mask_pan_relaxed(gen_rnd_pan()) |
    +---------------------------------+
    | 398403XXXXXX9547                |
    +---------------------------------+
    mysql> SELECT mask_pan_relaxed(gen_rnd_pan(19));
    +-----------------------------------+
    | mask_pan_relaxed(gen_rnd_pan(19)) |
    +-----------------------------------+
    | 578416XXXXXXXXX6509               |
    +-----------------------------------+
    mysql> SELECT gen_rnd_pan(11), gen_rnd_pan(20);
    +-----------------+-----------------+
    | gen_rnd_pan(11) | gen_rnd_pan(20) |
    +-----------------+-----------------+
    | NULL            | NULL            |
    +-----------------+-----------------+
    ```

+   `gen_rnd_ssn()`

    以`*`AAA`*-*`BB`*-*`CCCC`*`格式生成一个随机的美国社会安全号码。*`AAA`*部分大于 900，*`BB`*部分小于 70，这些特征不用于合法的社会安全号码。

    参数：

    无。

    返回值：

    一个随机的社会安全号码作为一个字符串。

    示例：

    ```sql
    mysql> SELECT gen_rnd_ssn();
    +---------------+
    | gen_rnd_ssn() |
    +---------------+
    | 951-26-0058   |
    +---------------+
    ```

+   `gen_rnd_us_phone()`

    以`1-555-*`AAA`*-*`BBBB`*`格式生成一个随机的美国电话号码。555 区号不用于合法的电话号码。

    参数：

    无。

    返回值：

    一个随机的美国电话号码作为一个字符串。

    示例：

    ```sql
    mysql> SELECT gen_rnd_us_phone();
    +--------------------+
    | gen_rnd_us_phone() |
    +--------------------+
    | 1-555-682-5423     |
    +--------------------+
    ```

##### 基于随机数据字典的插件函数

本节中的插件函数操作术语字典，并根据它们执行生成和掩盖操作。其中一些函数需要`SUPER`权限。

当加载字典时，它成为字典注册表的一部分，并被分配一个名称供其他字典函数使用。字典是从包含每行一个术语的纯文本文件中加载的。空行将被忽略。为了有效，字典文件必须至少包含一行非空行。

+   `gen_blacklist(*`str`*, *`dictionary_name`*, *`replacement_dictionary_name`*)`

    用第二个字典中的术语替换第一个字典中存在的术语，并返回替换术语。这通过替换掩盖了原始术语。此函数在 MySQL 8.0.23 中已弃用；请改用`gen_blocklist()`。

+   `gen_blocklist(*`str`*, *`dictionary_name`*, *`replacement_dictionary_name`*)`

    用第二个字典中的术语替换第一个字典中存在的术语，并返回替换术语。这通过替换掩盖了原始术语。此函数在 MySQL 8.0.23 中添加；请改用`gen_blacklist()`。

    参数：

    +   *`str`*：指示要替换的术语的字符串。

    +   *`dictionary_name`*：命名包含要替换的术语的字典的字符串。

    +   *`replacement_dictionary_name`*：命名要选择替换术语的字典的字符串。

    返回值：

    从*`replacement_dictionary_name`*中随机选择的字符串作为*`str`*的替换，如果它不在*`dictionary_name`*中，则为*`str`*，如果任一字典名称不在字典注册表中，则为`NULL`。

    如果要替换的术语出现在两个字典中，返回值可能是相同的术语。

    示例：

    ```sql
    mysql> SELECT gen_blocklist('Berlin', 'DE_Cities', 'US_Cities');
    +---------------------------------------------------+
    | gen_blocklist('Berlin', 'DE_Cities', 'US_Cities') |
    +---------------------------------------------------+
    | Phoenix                                           |
    +---------------------------------------------------+
    ```

+   `gen_dictionary(*`dictionary_name`*)`

    从字典中返回一个随机术语。

    参数：

    +   *`dictionary_name`*：命名要选择术语的字典的字符串。

    返回值：

    从字典中随机选择一个术语作为字符串，如果字典名称不在字典注册表中，则返回`NULL`。

    示例：

    ```sql
    mysql> SELECT gen_dictionary('mydict');
    +--------------------------+
    | gen_dictionary('mydict') |
    +--------------------------+
    | My term                  |
    +--------------------------+
    mysql> SELECT gen_dictionary('no-such-dict');
    +--------------------------------+
    | gen_dictionary('no-such-dict') |
    +--------------------------------+
    | NULL                           |
    +--------------------------------+
    ```

+   `gen_dictionary_drop(*`dictionary_name`*)`

    从字典注册表中移除一个字典。

    此函数需要`SUPER`权限。

    参数：

    +   *`dictionary_name`*：一个字符串，用于指定要从字典注册表中移除的字典的名称。

    返回值：

    一个指示删除操作是否成功的字符串。`Dictionary removed`表示成功。`Dictionary removal error`表示失败。

    示例：

    ```sql
    mysql> SELECT gen_dictionary_drop('mydict');
    +-------------------------------+
    | gen_dictionary_drop('mydict') |
    +-------------------------------+
    | Dictionary removed            |
    +-------------------------------+
    mysql> SELECT gen_dictionary_drop('no-such-dict');
    +-------------------------------------+
    | gen_dictionary_drop('no-such-dict') |
    +-------------------------------------+
    | Dictionary removal error            |
    +-------------------------------------+
    ```

+   `gen_dictionary_load(*`dictionary_path`*, *`dictionary_name`*)`

    将文件加载到字典注册表中，并为字典分配一个名称，以便在其他需要字典名称参数的函数中使用。

    此函数需要`SUPER`权限。

    重要提示

    字典不是持久的。应用程序使用的任何字典都必须在每次服务器启动时加载。

    一旦加载到注册表中，字典将按原样使用，即使底层字典文件发生更改。要重新加载字典，首先使用`gen_dictionary_drop()`将其删除，然后再次使用`gen_dictionary_load()`加载。

    参数：

    +   *`dictionary_path`*：一个字符串，指定字典文件的路径名。

    +   *`dictionary_name`*：一个字符串，为字典提供名称。

    返回值：

    一个指示加载操作是否成功的字符串。`Dictionary load success`表示成功。`Dictionary load error`表示失败。字典加载失败可能出现多种原因，包括：

    +   已加载具有给定名称的字典。

    +   未找到字典文件。

    +   字典文件不包含任何术语。

    +   `secure_file_priv`系统变量已设置，但字典文件未位于该变量指定的目录中。

    示例：

    ```sql
    mysql> SELECT gen_dictionary_load('/usr/local/mysql/mysql-files/mydict','mydict');
    +---------------------------------------------------------------------+
    | gen_dictionary_load('/usr/local/mysql/mysql-files/mydict','mydict') |
    +---------------------------------------------------------------------+
    | Dictionary load success                                             |
    +---------------------------------------------------------------------+
    mysql> SELECT gen_dictionary_load('/dev/null','null');
    +-----------------------------------------+
    | gen_dictionary_load('/dev/null','null') |
    +-----------------------------------------+
    | Dictionary load error                   |
    +-----------------------------------------+
    ```
