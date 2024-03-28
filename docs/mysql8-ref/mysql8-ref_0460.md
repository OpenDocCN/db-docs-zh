> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-masking-component-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/data-masking-component-functions.html)

#### 8.5.2.4 MySQL 企业数据掩码和去标识化组件功能描述

MySQL 企业数据掩码和去标识化组件包括几个函数，可以分为以下类别：

+   数据掩码组件函数

+   生成随机数据组件函数

+   字典掩码管理组件函数

+   生成字典组件函数

##### 数据掩码组件函数

本节中的每个组件函数对其字符串参数执行掩码操作并返回掩码结果。

+   [`mask_canada_sin(*`str`* [, *`mask_char`*])`](data-masking-component-functions.html#function_mask-canada-sin)

    掩盖加拿大社会保险号（SIN）并返回所有有意义的数字被`'X'`字符替换的数字。可以指定一个可选的掩盖字符。

    参数：

    +   *`str`*: 需要掩盖的字符串。接受的格式有：

        +   九个不间隔的数字。

        +   九个数字按照模式分组：`xxx-xxx-xxx`（'`-`'是任何分隔字符）。

        此参数将转换为`utf8mb4`字符集。

    +   *`mask_char`*: （可选）用于掩盖的单个字符。如果未提供*`mask_char`*，默认为`'X'`。

    返回值：

    掩盖后的加拿大社会保险号作为一个字符串，编码为`utf8mb4`字符集，如果参数长度不正确则报错，或者如果*`str`*格式不正确或包含多字节字符则返回`NULL`。

    示例：

    ```sql
    mysql> SELECT mask_canada_sin('046-454-286'), mask_canada_sin('abcdefijk');
    +--------------------------------+------------------------------+
    | mask_canada_sin('046-454-286') | mask_canada_sin('abcdefijk') |
    +--------------------------------+------------------------------+
    | XXX-XXX-XXX                    | XXXXXXXXX                    |
    +--------------------------------+------------------------------+
    mysql> SELECT mask_canada_sin('909');
    ERROR 1123 (HY000): Can't initialize function 'mask_canada_sin'; Argument 0 is too short.
    mysql> SELECT mask_canada_sin('046-454-286-909');
    ERROR 1123 (HY000): Can't initialize function 'mask_canada_sin'; Argument 0 is too long.
    ```

+   [`mask_iban(*`str`* [, *`mask_char`*])`](data-masking-component-functions.html#function_mask-iban)

    掩盖国际银行账号（IBAN）并返回除了前两个字母（表示国家）以外的所有字符被`'*'`字符替换的数字。可以指定一个可选的掩盖字符。

    参数：

    +   *`str`*: 需要掩盖的字符串。每个国家可能有不同的国家路由或账号编号系统，最少 13 个字符，最多 34 个字母数字 ASCII 字符。接受的格式有：

        +   非分隔字符。

        +   每四个字符分组，除了最后一组，用空格或其他分隔字符分隔（例如：`xxxx-xxxx-xxxx-xx`）。

        此参数将转换为`utf8mb4`字符集。

    +   *`mask_char`*: （可选）用于掩盖的单个字符。如果未提供*`mask_char`*，默认为`'*'`。

    返回值：

    作为字符串编码在`utf8mb4`字符集中的掩码国际银行账号，如果参数长度不正确则返回错误，如果*`str`*格式不正确或包含多字节字符则返回`NULL`。

    示例：

    ```sql
    mysql> SELECT mask_iban('IE12 BOFI 9000 0112 3456 78'), mask_iban('abcdefghijk');
    +------------------------------------------+--------------------------+
    | mask_iban('IE12 BOFI 9000 0112 3456 78') | mask_iban('abcdefghijk') |
    +------------------------------------------+--------------------------+
    | IE** **** **** **** **** **              | ab*********              |
    +------------------------------------------+--------------------------+
    mysql> SELECT mask_iban('909');
    ERROR 1123 (HY000): Can't initialize function 'mask_iban'; Argument 0 is too short.
    mysql> SELECT mask_iban('IE12 BOFI 9000 0112 3456 78 IE12 BOFI 9000 0112 3456 78');
    ERROR 1123 (HY000): Can't initialize function 'mask_iban'; Argument 0 is too long.
    ```

+   [`mask_inner(*`str`*, *`margin1`*, *`margin2`* [, *`mask_char`*])`](data-masking-component-functions.html#function_mask-inner)

    掩码字符串的内部部分，保留两端不变，并返回结果。可以指定一个可选的掩码字符。

    `mask_inner`支持所有字符集。

    参数：

    +   *`str`*：要掩码的字符串。此参数将转换为`utf8mb4`字符集。

    +   *`margin1`*：一个非负整数，指定要保留不掩码的字符串左端字符的数量。如果值为 0，则不保留任何左端字符。

    +   *`margin2`*：一个非负整数，指定要保留不掩码的字符串右端字符的数量。如果值为 0，则不保留任何右端字符。

    +   *`mask_char`*：（可选）用于掩码的单个字符。如果未提供*`mask_char`*，默认为`'X'`。

    返回值：

    使用与*`str`*相同的字符集编码的掩码字符串，如果任一边界为负则返��错误。

    如果边界值的总和大于参数长度，则不进行掩码操作，直接返回参数。

    注意

    该函数经过优化，可更快地处理单字节字符串（具有相等的字节长度和字符长度）。例如，`utf8mb4`字符集仅使用一个字节表示 ASCII 字符，因此该函数将只处理包含 ASCII 字符的字符串作为单字节字符字符串。

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

+   [`mask_outer(*`str`*, *`margin1`*, *`margin2`* [, *`mask_char`*])`](data-masking-component-functions.html#function_mask-outer)

    掩码字符串的左端和右端，保留内部不变，并返回结果。可以指定一个可选的掩码字符。

    `mask_outer`支持所有字符集。

    参数：

    +   *`str`*：要掩码的字符串。此参数将转换为`utf8mb4`字符集。

    +   *`margin1`*：一个非负整数，指定要掩码的字符串左端字符的数量。如果值为 0，则不对左端字符进行掩码。

    +   *`margin2`*：一个非负整数，指定要掩码的字符串右端字符的数量。如果值为 0，则不对右端字符进行掩码。

    +   *`mask_char`*：（可选）用于掩码的单个字符。如果未提供*`mask_char`*，默认为`'X'`。

    返回值：

    使用与*`str`*相同的字符集编码的掩码字符串，如果任一边界为负则返回错误。

    如果边界值的总和大于参数长度，则整个参数都会被掩码。

    注意

    该函数经过优化，以便更快地处理单字节字符串（具有相等的字节长度和字符长度）。例如，`utf8mb4`字符集仅使用一个字节来表示 ASCII 字符，因此该函数将仅处理包含 ASCII 字符的字符串作为单字节字符字符串。

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

+   [`mask_pan(*`str`* [, *`mask_char`*])`](data-masking-component-functions.html#function_mask-pan)

    掩码一个支付卡的主帐号（PAN），并返回除了最后四位数字以外的所有数字都替换为`'X'`字符的号码。可以指定一个可选的掩码字符。

    参数：

    +   *`str`*：要掩码的字符串。该字符串必须包含最少 14 个最多 19 个字母数字字符。此参数被转换为`utf8mb4`字符集。

    +   *`mask_char`*：（可选）用于掩码的单个字符。如果未提供*`mask_char`*，默认为`'X'`。

    返回值：

    掩码后的支付号码作为以`utf8mb4`字符集编码的字符串，如果参数长度不正确则报错，或者如果*`str`*格式不正确或包含多字节字符则返回`NULL`。

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
    ERROR 1123 (HY000): Can't initialize function 'mask_pan'; Argument 0 is too short.
    ```

+   `mask_pan_relaxed(*`str`*)`

    掩码一个支付卡的主帐号，并返回除了前六位和最后四位数字以外的所有数字都替换为`'X'`字符的号码。前六位数字表示支付卡发行者。可以指定一个可选的掩码字符。

    参数：

    +   *`str`*：要掩��的字符串。该字符串必须适合主帐号号码的长度，但不进行其他检查。此参数被转换为`utf8mb4`字符集。

    +   *`mask_char`*：（可选）用于掩码的单个字符。如果未提供*`mask_char`*，默认为`'X'`。

    返回值：

    掩码后的支付号码作为以`utf8mb4`字符集编码的字符串，如果参数长度不正确则报错，或者如果*`str`*格式不正确或包含多字节字符则返回`NULL`。

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
    ERROR 1123 (HY000): Can't initialize function 'mask_pan_relaxed'; Argument 0 is too short.
    ```

+   [`mask_ssn(*`str`* [, *`mask_char`*])`](data-masking-component-functions.html#function_mask-ssn)

    掩码一个美国社会安全号码（SSN），并返回除了最后四位数字以外的所有数字都替换为`'*'`字符。可以指定一个可选的掩码字符。

    参数：

    +   *`str`*：要掩码的字符串。接受的格式为：

        +   九个不间断的数字。

        +   九位数字按照模式分组：`xxx-xx-xxxx`（'`-`'是任何分隔符字符）。

        这个参数被转换为`utf8mb4`字符集。

    +   *`mask_char`*：（可选）用于掩码的单个字符。如果未提供*`mask_char`*，默认为`'*'`。

    返回值：

    掩码后的社会安全号码作为以`utf8mb4`字符集编码的字符串，如果参数长度不正确则报错，或者如果*`str`*格式不正确或包含多字节字符则返回`NULL`。

    示例：

    ```sql
    mysql> SELECT mask_ssn('909-63-6922'), mask_ssn('cdefghijk');
    +-------------------------+-------------------------+
    | mask_ssn('909-63-6922') | mask_ssn('cdefghijk')   |
    +-------------------------+-------------------------+
    | ***-**-6922             | *******hijk             |
    +-------------------------+-------------------------+
    mysql> SELECT mask_ssn('909');
    ERROR 1123 (HY000): Can't initialize function 'mask_ssn'; Argument 0 is too short.
    mysql> SELECT mask_ssn('123456789123456789');
    ERROR 1123 (HY000): Can't initialize function 'mask_ssn'; Argument 0 is too long.
    ```

+   [`mask_uk_nin(*`str`* [, *`mask_char`*])`](data-masking-component-functions.html#function_mask-uk-nin)

    掩码一个英国国民保险号码（UK NIN）并返回除了前两位数字外所有字符都被`'*'`字符替换的数字��可以指定一个可选的掩码字符。

    参数：

    +   *`str`*：要掩码的字符串。接受的格式有：

        +   九个非分隔数字。

        +   以模式分组的九个数字：`xxx-xx-xxxx`（'`-`'是任何分隔符字符）。

        +   以模式分组的九个数字：`xx-xxxxxx-x`（'`-`'是任何分隔符字符）。

        此参数转换为`utf8mb4`字符集。

    +   *`mask_char`*：（可选）用于掩码的单个字符。如果未提供*`mask_char`*，则默认为`'*'`。

    返回值：

    作为字符串编码在`utf8mb4`字符集中的掩码英国国民保险号码，如果参数长度不正确则报错，或者如果*`str`*格式不正确或包含多字节字符则返回`NULL`。

    示例：

    ```sql
    mysql> SELECT mask_uk_nin('QQ 12 34 56 C'), mask_uk_nin('abcdefghi');
    +------------------------------+--------------------------+
    | mask_uk_nin('QQ 12 34 56 C') | mask_uk_nin('abcdefghi') |
    +------------------------------+--------------------------+
    | QQ ** ** ** *                | ab*******                |
    +------------------------------+--------------------------+
    mysql> SELECT mask_uk_nin('909');
    ERROR 1123 (HY000): Can't initialize function 'mask_uk_nin'; Argument 0 is too short.
    mysql> SELECT mask_uk_nin('abcdefghijk');
    ERROR 1123 (HY000): Can't initialize function 'mask_uk_nin'; Argument 0 is too long.
    ```

+   [`mask_uuid(*`str`* [, *`mask_char`*])`](data-masking-component-functions.html#function_mask-uuid)

    掩码一个通用唯一标识符（UUID）并返回所有有意义字符被`'*'`字符替换的数字。可以指定一个可选的掩码字符。

    参数：

    +   *`str`*：要掩码的字符串。接受的格式为`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`，其中'`X`'是任何数字，'`-`'是任何分隔符字符。此参数转换为`utf8mb4`字符集。

    +   *`mask_char`*：（可选）用于掩码的单个字符。如果未提供*`mask_char`*，则默认为`'*'`。

    返回值：

    作为字符串编码在`utf8mb4`字符集中的掩码 UUID，如果参数长度不正确则报错，或者如果*`str`*格式不正确或包含多字节字符则返回`NULL`。

    示例：

    ```sql
    mysql> SELECT mask_uuid(gen_rnd_uuid());
    +--------------------------------------+
    | mask_uuid(gen_rnd_uuid())            |
    +--------------------------------------+
    | ********-****-****-****-************ |
    +--------------------------------------+
    mysql> SELECT mask_uuid('909');
    ERROR 1123 (HY000): Can't initialize function 'mask_uuid'; Argument 0 is too short.
    mysql> SELECT mask_uuid('123e4567-e89b-12d3-a456-426614174000-123e4567-e89b-12d3');
    ERROR 1123 (HY000): Can't initialize function 'mask_uuid'; Argument 0 is too long.
    ```

##### 随机数据生成组件函数

本节中的组件函数为不同类型的数据生成随机值。在可能的情况下，生成的值具有保留用于演示或测试值的特征，以避免将它们误认为是合法数据。例如，`gen_rnd_us_phone()`返回一个使用未分配给实际使用的电话号码的 555 区号的美国电话号码。各个函数的描述会说明任何违反此原则的情况。

+   `gen_range(*`lower`*, *`upper`*)`

    生成从指定范围中选择的随机数。

    参数：

    +   *`lower`*：指定范围的下限的整数。

    +   *`upper`*：指定范围的上限，必须不小于下限。

    返回值：

    从*`lower`*到*`upper`*范围内的随机整数（以`utf8mb4`字符集编码），包括*`lower`*和*`upper`*，如果*`upper`*参数小于*`lower`*则返回`NULL`。

    注意

    为了获得更好质量的随机值，请使用`RAND()`而不是此函数。

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

+   `gen_rnd_canada_sin()`

    以`*`AAA`*-*`BBB`*-*`CCC`*`格式生成一个随机的加拿大社会保险号码。生成的号码通过 Luhn 检查算法，确保此号码的一致性。

    警告

    从`gen_rnd_canada_sin()`返回的值仅用于测试目的，不适合发布。无法保证给定的返回值不会分配给合法的加拿大社会保险号码。如果需要发布`gen_rnd_canada_sin()`的结果，请考虑使用`mask_canada_sin()`进行掩码处理。

    参数:

    无。

    返回值:

    一个以`utf8mb4`字符集编码的随机加拿大社会保险号码。

    示例:

    ```sql
    mysql> SELECT gen_rnd_canada_sin();
    +----------------------+
    | gen_rnd_canada_sin() |
    +----------------------+
    | 046-454-286          |
    +----------------------+
    ```

+   `gen_rnd_email(*`name_size`*, *`surname_size`*, *`domain`*)`

    生成一个以*`random_name`*.*`random_surname`*@*`domain`*形式的随机电子邮件地址。

    参数:

    +   *`name_size`*: （可选）一个整数，指定地址中名称部分的字符数。如果未提供*`name_size`*，默认值为五。

    +   *`surname_size`*: （可选）一个整数，指定地址中姓氏部分的字符数。如果未提供*`surname_size`*，默认值为七。

    +   *`domain`*: （可选）一个字符串，指定地址的域部分。如果未提供*`domain`*，默认值为`example.com`。

    返回值:

    一个以`utf8mb4`字符集编码的随机电子邮件地址。

    示例:

    ```sql
    mysql> SELECT gen_rnd_email(name_size = 4, surname_size = 5, domain = 'mynet.com');
    +----------------------------------------------------------------------+
    | gen_rnd_email(name_size = 4, surname_size = 5, domain = 'mynet.com') |
    +----------------------------------------------------------------------+
    | lsoy.qwupp@mynet.com                                                 |
    +----------------------------------------------------------------------+
    mysql> SELECT gen_rnd_email();
    +---------------------------+
    | gen_rnd_email()           |
    +---------------------------+
    | ijocv.mwvhhuf@example.com |
    +---------------------------+
    ```

+   [`gen_rnd_iban([*`country`*, *`size`*])`](data-masking-component-functions.html#function_gen-rnd-iban)

    以`*`AAAA`* *`BBBB`* *`CCCC`* *`DDDD`*`格式生成一个随机的国际银行账号（IBAN）。生成的字符串以两位字符的国家代码开头，根据 IBAN 规范计算的两位校验数字和随机的字母数字字符，直到达到所需大小。

    警告

    从`gen_rnd_iban()`返回的值仅用于测试目的，如果与有效的国家代码一起使用，则不适合发布。无法保证给定的返回值不会分配给合法的银行账户。如果需要发布`gen_rnd_iban()`的结果，请考虑使用`mask_iban()`进行掩码处理。

    参数:

    +   *`country`*: （可选）两位字符的国家代码；默认值为`ZZ`

    +   *`size`*: （可选）有意义字符的数量；默认 16，最小 15，最大 34

    返回值:

    以`utf8mb4`字符集编码的随机 IBAN。

    示例:

    ```sql
    mysql> SELECT gen_rnd_iban();
    +-----------------------------+
    | gen_rnd_iban()              |
    +-----------------------------+
    | ZZ79 3K2J WNH9 1V0DI        |
    +-----------------------------+
    ```

+   [`gen_rnd_pan([*`size`*])`](data-masking-component-functions.html#function_gen-rnd-pan)

    生成一个随机的支付卡主帐号。该号码通过 Luhn 检查（执行校验和验证以对检查位进行检查的算法）。

    警告

    从`gen_rnd_pan()`返回的数值仅应用于测试目的，不适合发布。无法保证给定的返回值不会分配给合法的支付账户。如果需要发布`gen_rnd_pan()`的结果，考虑使用`mask_pan()`或`mask_pan_relaxed()`进行掩码处理。

    参数：

    +   *`size`*：（可选）一个指定结果大小的整数。如果未给出*`size`*，默认值为 16。如果给出，*`size`*必须是 12 到 19 范围内的整数。

    返回值：

    作为字符串的随机支付号码，或者如果给定超出允许范围的*`size`*参数，则返回错误。

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
    mysql> SELECT gen_rnd_pan(20);
    ERROR 1123 (HY000): Can't initialize function 'gen_rnd_pan'; Minimal value of argument 0 is 14.
    ```

+   `gen_rnd_ssn()`

    以`*`AAA`*-*`BB`*-*`CCCC`*`格式生成一个随机的美国社会保障号码。*`AAA`*部分大于 900，这些特征不用于合法的社会保障号码。

    参数：

    无。

    返回值：

    作为以`utf8mb4`字符集编码的字符串的随机社会保障号码。

    示例：

    ```sql
    mysql> SELECT gen_rnd_ssn();
    +---------------+
    | gen_rnd_ssn() |
    +---------------+
    | 951-26-0058   |
    +---------------+
    ```

+   `gen_rnd_uk_nin()`

    生成一个随机的英国国民保险号码（UK NIN），格式为九个字符。NIN 以从有效前缀集合中随机选择的两个字符前缀开始，接着是六个随机数字，以及从有效后缀集合中随机选择的一个字符后缀。

    警告

    从`gen_rnd_uk_nin()`返回的数值仅应用于测试目的，不适合发布。无法保证给定的返回值不会分配给合法的 NIN。如果需要发布`gen_rnd_uk_nin()`的结果，考虑使用`mask_uk_nin()`进行掩码处理。

    参数：

    无。

    返回值：

    作为以`utf8mb4`字符集编码的字符串的随机英国国民保险号码。

    示例：

    ```sql
    mysql> SELECT gen_rnd_uk_nin();
    +----------------------+
    | gen_rnd_uk_nin()     |
    +----------------------+
    | QQ123456C            |
    +----------------------+
    ```

+   `gen_rnd_us_phone()`

    以`1-555-*`AAA`*-*`BBBB`*`格式生成一个随机的美国电话号码。555���号不用于合法的电话号码。

    参数：

    无。

    返回值：

    作为以`utf8mb4`字符集编码的字符串的随机美国电话号码。

    示例：

    ```sql
    mysql> SELECT gen_rnd_us_phone();
    +--------------------+
    | gen_rnd_us_phone() |
    +--------------------+
    | 1-555-682-5423     |
    +--------------------+
    ```

+   `gen_rnd_uuid()`

    生成带有破折号分隔的随机通用唯一标识符（UUID）。

    参数：

    无。

    返回值：

    作为`utf8mb4`字符集中编码的字符串的随机 UUID。

    示例：

    ```sql
    mysql> SELECT gen_rnd_uuid();
    +--------------------------------------+
    | gen_rnd_uuid()                       |
    +--------------------------------------+
    | 123e4567-e89b-12d3-a456-426614174000 |
    +--------------------------------------+
    ```

##### 字典掩码管理组件函数

本节中的组件函数操作术语字典并根据其执行管理掩码操作。所有这些函数都需要`MASKING_DICTIONARIES_ADMIN`权限。

创建术语字典时，它将成为字典注册表的一部分，并被分配一个名称供其他字典函数使用。

+   `masking_dictionary_remove(*`dictionary_name`*)`

    从字典注册表中删除一个字典及其所有术语。此函数需要`MASKING_DICTIONARIES_ADMIN`权限。

    参数：

    +   *`dictionary_name`*：命名要从字典表中移除的字典的字符串。此参数转换为`utf8mb4`字符集。

    返回值：

    一个字符串，指示移除操作是否成功。`1`表示成功。`NULL`表示未找到字典名称。

    示例：

    ```sql
    mysql> SELECT masking_dictionary_remove('mydict');
    +-------------------------------------+
    | masking_dictionary_remove('mydict') |
    +-------------------------------------+
    |                                   1 |
    +-------------------------------------+
    mysql> SELECT masking_dictionary_remove('no-such-dict');
    +-------------------------------------------+
    | masking_dictionary_remove('no-such-dict') |
    +-------------------------------------------+
    |                                      NULL |
    +-------------------------------------------+
    ```

+   `masking_dictionary_term_add(*`dictionary_name`*, *`term_name`*)`

    向命名字典添加一个术语。此函数需要`MASKING_DICTIONARIES_ADMIN`权限。

    重要

    字典及其术语持久保存在`mysql`模式的表中。如果用户重复执行`gen_dictionary()`，则字典中的所有术语对任何用户帐户都是可访问的。避免向字典中添加敏感信息。

    每个术语由一个命名的字典定义。`masking_dictionary_term_add()`允许您一次添加一个字典术语。

    参数：

    +   *`dictionary_name`*：为字典提供名称的字符串。此参数转换为`utf8mb4`字符集。

    +   *`term_name`*：指定字典表中术语名称的字符串。此参数转换为`utf8mb4`字符集。

    返回值：

    一个字符串，指示添加术语操作是否成功。`1`表示成功。`NULL`表示失败。术语添加失败可能出现多种原因，包括：

    +   已经添加了具有给定名称的术语。

    +   未找到字典名称。

    示例：

    ```sql
    mysql> SELECT masking_dictionary_term_add('mydict','newterm');
    +-------------------------------------------------+
    | masking_dictionary_term_add('mydict','newterm') |
    +-------------------------------------------------+
    |                                               1 |
    +-------------------------------------------------+
    mysql> SELECT masking_dictionary_term_add('mydict','');
    +------------------------------------------+
    | masking_dictionary_term_add('mydict','') |
    +------------------------------------------+
    |                                     NULL |
    +------------------------------------------+
    ```

+   `masking_dictionary_term_remove(*`dictionary_name`*, *`term_name`*)`

    从命名字典中删除一个术语。此函数需要`MASKING_DICTIONARIES_ADMIN`权限。

    参数：

    +   *`dictionary_name`*: 为字典提供名称的字符串。此参数转换为`utf8mb4`字符集。

    +   *`term_name`*: 指定字典表中术语名称的字符串。此参数转换为`utf8mb4`字符集。

    返回值：

    一个指示删除术语操作是否成功的字符串。`1`表示成功。`NULL`表示失败。术语删除失败可能出现多种原因，包括：

    +   未找到具有给定名称的术语。

    +   未找到字典名称。

    示例：

    ```sql
    mysql> SELECT masking_dictionary_term_add('mydict','newterm');
    +-------------------------------------------------+
    | masking_dictionary_term_add('mydict','newterm') |
    +-------------------------------------------------+
    |                                               1 |
    +-------------------------------------------------+
    mysql> SELECT masking_dictionary_term_remove('mydict','');
    +---------------------------------------------+
    | masking_dictionary_term_remove('mydict','') |
    +---------------------------------------------+
    |                                        NULL |
    +---------------------------------------------+
    ```

##### 字典生成组件函数

本节中的组件函数操作术语字典并基于其执行生成操作。

创建术语字典时，它将成为字典注册表的一部分，并被分配一个名称供其他字典函数使用。

+   `gen_blocklist(*`str`*, *`from_dictionary_name`*, *`to_dictionary_name`*)`

    用第二个字典中的术语替换第一个字典中的术语，并返回替换的术语。这通过替换掩盖了原始术语。

    参数：

    +   *`term`*: 指示要替换的术语的字符串。此参数转换为`utf8mb4`字符集。

    +   *`from_dictionary_name`*: 命名包含要替换的术语的字典的字符串。此参数转换为`utf8mb4`字符集。

    +   *`to_dictionary_name`*: 命名要从中选择替换术语的字典的字符串。此参数转换为`utf8mb4`字符集。

    返回值：

    从*`to_dictionary_name`*中随机选择的以`utf8mb4`字符集编码的字符串作为*`term`*的替换，如果*`term`*不在*`from_dictionary_name`*中出现，则返回*`term`*，或者如果任一字典名称不在字典注册表中，则返回错误。

    注意

    如果要替换的术语出现在两个字典中，则返回值可能是相同的术语。

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

    返回一个字典中的随机术语。

    参数：

    +   *`dictionary_name`*: 命名要从中选择术语的字典的字符串。此参数转换为`utf8mb4`字符集。

    返回值：

    从字典中以`utf8mb4`字符集编码的字符串中返回一个随机术语，如果字典名称不在字典注册表中，则返回`NULL`。

    示例：

    ```sql
    mysql> SELECT gen_dictionary('mydict');
    +--------------------------+
    | gen_dictionary('mydict') |
    +--------------------------+
    | My term                  |
    +--------------------------+
    mysql> SELECT gen_dictionary('no-such-dict');
    ERROR 1123 (HY000): Can't initialize function 'gen_dictionary'; Cannot access 
    dictionary, check if dictionary name is valid.
    ```
