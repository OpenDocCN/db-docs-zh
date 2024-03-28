# 13.3.1 字符串数据类型语法

> 原文：[`dev.mysql.com/doc/refman/8.0/en/string-type-syntax.html`](https://dev.mysql.com/doc/refman/8.0/en/string-type-syntax.html)

字符串数据类型包括`CHAR`, `VARCHAR`, `BINARY`, `VARBINARY`, `BLOB`, `TEXT`, `ENUM`, 和 `SET`。

在某些情况下，MySQL 可能会将字符串列更改为与`CREATE TABLE`或`ALTER TABLE`语句中给出的类型不同的类型。请参阅第 15.1.20.7 节，“静默列规范更改”。

对于字符串列的定义(`CHAR`, `VARCHAR`, 和 `TEXT`类型), MySQL 以字符单位解释长度规范。对于二进制字符串列的定义(`BINARY`, `VARBINARY`, 和 `BLOB`类型), MySQL 以字节单位解释长度规范。

字符串数据类型`CHAR`, `VARCHAR`, `TEXT`类型, `ENUM`, `SET`, 和任何同义词的列定义可以指定列字符集和排序规则：

+   `CHARACTER SET`指定字符集。如果需要，可以使用`COLLATE`属性指定字符集的排序规则，以及任何其他属性。例如：

    ```sql
    CREATE TABLE t
    (
        c1 VARCHAR(20) CHARACTER SET utf8mb4,
        c2 TEXT CHARACTER SET latin1 COLLATE latin1_general_cs
    );
    ```

    此表定义创建一个名为`c1`的列，其字符集为`utf8mb4`，具有该字符集的默认排序规则，并创建一个名为`c2`的列，其字符集为`latin1`，具有区分大小写(`_cs`)的排序规则。

    当`CHARACTER SET`和`COLLATE`属性中的一个或两个缺失时，关于分配字符集和排序规则的规则在第 12.3.5 节，“列字符集和排序”中描述。

    `CHARSET`是`CHARACTER SET`的同义词。

+   为字符串数据类型指定 `CHARACTER SET binary` 属性会导致该列被创建为相应的二进制字符串数据类型：`CHAR` 变为 `BINARY`，`VARCHAR` 变为 `VARBINARY`，`TEXT` 变为 `BLOB`。对于 `ENUM` 和 `SET` 数据类型，不会发生这种情况；它们将按声明创建。假设您使用以下定义指定一个表：

    ```sql
    CREATE TABLE t
    (
      c1 VARCHAR(10) CHARACTER SET binary,
      c2 TEXT CHARACTER SET binary,
      c3 ENUM('a','b','c') CHARACTER SET binary
    );
    ```

    结果表的定义如下：

    ```sql
    CREATE TABLE t
    (
      c1 VARBINARY(10),
      c2 BLOB,
      c3 ENUM('a','b','c') CHARACTER SET binary
    );
    ```

+   `BINARY` 属性是 MySQL 的非标准扩展，是指定列字符集的二进制 (`_bin`) 校对的简写（如果未指定列字符集，则是表默认字符集的二进制校对）。在这种情况下，比较和排序是基于数字字符代码值的。假设您使用以下定义指定一个表：

    ```sql
    CREATE TABLE t
    (
      c1 VARCHAR(10) CHARACTER SET latin1 BINARY,
      c2 TEXT BINARY
    ) CHARACTER SET utf8mb4;
    ```

    结果表的定义如下：

    ```sql
    CREATE TABLE t (
      c1 VARCHAR(10) CHARACTER SET latin1 COLLATE latin1_bin,
      c2 TEXT CHARACTER SET utf8mb4 COLLATE utf8mb4_bin
    ) CHARACTER SET utf8mb4;
    ```

    在 MySQL 8.0 中，`BINARY` 属性的这种非标准用法是模棱两可的，因为 `utf8mb4` 字符集有多个 `_bin` 校对。从 MySQL 8.0.17 开始，`BINARY` 属性已被弃用，您应该期望在将来的 MySQL 版本中删除对其的支持。应用程序应调整为使用显式的 `_bin` 校对。

    使用 `BINARY` 指定数据类型或字符集的用法保持不变。

+   `ASCII` 属性是 `CHARACTER SET latin1` 的简写。在较旧的 MySQL 版本中受支持，但在 MySQL 8.0.28 及更高版本中已被弃用；请使用 `CHARACTER SET` 替代。

+   `UNICODE` 属性是 `CHARACTER SET ucs2` 的简写。在较旧的 MySQL 版本中受支持，但在 MySQL 8.0.28 及更高版本中已被弃用；请使用 `CHARACTER SET` 替代。

字符列的比较和排序基于分配给列的校对规则。对于 `CHAR`、`VARCHAR`、`TEXT`、`ENUM` 和 `SET` 数据类型，您可以声明一个带有二进制 (`_bin`) 校对或 `BINARY` 属性的列，以使比较和排序使用底层字符代码值而不是词法顺序。

有关 MySQL 中字符集使用的更多信息，请参阅 第十二章，*字符集、校对规则、Unicode*。

+   `[NATIONAL] CHAR[(*`M`*)] [CHARACTER SET *`charset_name`*] [COLLATE *`collation_name`*]`

    固定长度的字符串，在存储时总是用空格右填充到指定长度。*`M`*代表列长度（以字符为单位）。*`M`*的范围是 0 到 255。如果省略*`M`*，长度为 1。

    注意

    当检索`CHAR`值时，尾随空格会被移除，除非启用了`PAD_CHAR_TO_FULL_LENGTH` SQL 模式。

    `CHAR`是`CHARACTER`的简写。`NATIONAL CHAR`（或其等效的简写形式，`NCHAR`）是定义`CHAR`列应使用某个预定义字符集的标准 SQL 方式。MySQL 使用`utf8mb3`作为这个预定义字符集。第 12.3.7 节，“国家字符集”。

    `CHAR BYTE`数据类型是`BINARY`数据类型的别名。这是一个兼容性特性。

    MySQL 允许创建类型为`CHAR(0)`的列。这在必须符合依赖于列存在但实际上不使用其值的旧应用程序时非常有用。当您需要一个只能取两个值的列时，`CHAR(0)`也非常好：定义为`CHAR(0) NULL`的列仅占用一个位，并且只能取`NULL`和`''`（空字符串）这两个值。

+   `[NATIONAL] VARCHAR(*`M`*) [字符集 *`charset_name`*] [校对 *`collation_name`*]`

    可变长度字符串。*`M`*表示最大列长度（以字符为单位）。*`M`*的范围是 0 到 65,535。`VARCHAR`的有效最大长度取决于最大行大小（65,535 字节，在所有列之间共享）和所使用的字符集。例如，`utf8mb3`字符可能每个字符需要最多三个字节，因此使用`utf8mb3`字符集的`VARCHAR`列最大可以声明为 21,844 个字符。参见第 10.4.7 节，“表列计数和行大小限制”。

    MySQL 将`VARCHAR`值存储为 1 字节或 2 字节长度前缀加数据。长度前缀指示值中的字节数。如果值不超过 255 字节，则`VARCHAR`列使用一个长度字节，如果值可能超过 255 字节，则使用两个长度字节。

    注意

    MySQL 遵循标准 SQL 规范，*不会*从 `VARCHAR` 值中删除尾随空格。

    `VARCHAR` 是 `CHARACTER VARYING` 的简写。`NATIONAL VARCHAR` 是定义 `VARCHAR` 列应使用某个预定义字符集的标准 SQL 方式。MySQL 使用 `utf8mb3` 作为这个预定义字符集。第 12.3.7 节，“国家字符集”。`NVARCHAR` 是 `NATIONAL VARCHAR` 的简写。

+   [`BINARY[(*`M`*)]`](binary-varbinary.html "13.3.3 BINARY 和 VARBINARY 类型")

    `BINARY` 类型类似于 `CHAR` 类型，但存储的是二进制字节字符串而不是非二进制字符字符串。可选长度 *`M`* 表示列长度（以字节为单位）。如果省略，*`M`* 默认为 1。

+   `VARBINARY(*`M`*)`

    `VARBINARY` 类型类似于 `VARCHAR` 类型，但存储的是二进制字节字符串而不是非二进制字符字符串。 *`M`* 表示最大列长度（以字节为单位）。

+   `TINYBLOB`

    最大长度为 255（2⁸ − 1）字节的 `BLOB` 列。每个 `TINYBLOB` 值都使用一个 1 字节长度前缀来存储，指示值中的字节数。

+   [`TINYTEXT [字符集 *`charset_name`*] [校对 *`collation_name`*]`](blob.html "13.3.4 BLOB 和 TEXT 类型")

    最大长度为 255（2⁸ − 1）个字符的 `TEXT` 列。如果值包含多字节字符，则有效最大长度会减少。每个 `TINYTEXT` 值都使用一个 1 字节长度前缀来存储，指示值中的字节数。

+   [`BLOB[(*`M`*)]`](blob.html "13.3.4 BLOB 和 TEXT 类型")

    最大长度为 65,535（2¹⁶ − 1）字节的 `BLOB` 列。每个 `BLOB` 值都使用一个 2 字节长度前缀来存储，指示值中的字节数。

    可以为此类型指定可选长度 *`M`*。如果这样做，MySQL 将创建一个足以容纳 *`M`* 字节长值的最小 `BLOB` 类型列。

+   [`TEXT[(*`M`*)] [字符集 *`charset_name`*] [校对 *`collation_name`*]`](blob.html "13.3.4 BLOB 和 TEXT 类型")

    一个最大长度为 65,535（2¹⁶ − 1）个字符的`TEXT`列。如果值包含多字节字符，则有效最大长度会更少。每个`TEXT`值都使用一个 2 字节长度前缀来指示值中的字节数。

    可以为此类型指定一个可选长度 *`M`*。如果这样做，MySQL 会创建一个足以容纳 *`M`* 个字符长的最小`TEXT`类型列。

+   `MEDIUMBLOB`

    一个最大长度为 16,777,215（2²⁴ − 1）字节的`BLOB`列。每个`MEDIUMBLOB`值都使用一个 3 字节长度前缀来指示值中的字节数。

+   [`MEDIUMTEXT [字符集 *`charset_name`*] [校对 *`collation_name`*]`](blob.html "13.3.4 BLOB 和 TEXT 类型")

    一个最大长度为 16,777,215（2²⁴ − 1）个字符的`TEXT`列。如果值包含多字节字符，则有效最大长度会更少。每个`MEDIUMTEXT`值都使用一个 3 字节长度前缀来指示值中的字节数。

+   `LONGBLOB`

    一个最大长度为 4,294,967,295 或 4GB（2³² − 1）字节的`BLOB`列。`LONGBLOB`列的有效最大长度取决于客户端/服务器协议中配置的最大数据包大小和可用内存。每个`LONGBLOB`值都使用一个 4 字节长度前缀来指示值中的字节数。

+   [`LONGTEXT [字符集 *`charset_name`*] [校对 *`collation_name`*]`](blob.html "13.3.4 BLOB 和 TEXT 类型")

    一个最大长度为 4,294,967,295 或 4GB（2³² − 1）个字符的`TEXT`列。如果值包含多字节字符，则有效最大长度会更少。`LONGTEXT`列的有效最大长度还取决于客户端/服务器协议中配置的最大数据包大小和可用内存。每个`LONGTEXT`值都使用一个 4 字节长度前缀来指示值中的字节数。

+   [`ENUM('*`value1`*','*`value2`*',...) [字符集 *`charset_name`*] [校对 *`collation_name`*]`](enum.html "13.3.5 ENUM 类型")

    一个枚举。一个字符串对象，只能有一个值，从值列表`'*`value1`*'`，`'*`value2`*'`，`...`，`NULL`或特殊的`''`错误值中选择。`ENUM`值在内部表示为整数。

    `ENUM`列最多可以有 65,535 个不同元素。

    单个`ENUM`元素的最大支持长度为*`M`* <= 255，且(*`M`* x *`w`*) <= 1020，其中`M`是元素文字长度，*`w`*是字符集中最大长度字符所需的字节数。

+   [`SET('*`value1`*','*`value2`*',...) [字符集 *`charset_name`*] [校对 *`collation_name`*]`](set.html "13.3.6 SET 类型")

    一个集合。一个字符串对象，可以有零个或多个值，每个值必须从值列表`'*`value1`*'`，`'*`value2`*'`，`...` `SET`中选择的值内部表示为整数。

    `SET`列最多可以有 64 个不同成员。

    单个`SET`元素的最大支持长度为*`M`* <= 255，且(*`M`* x *`w`*) <= 1020，其中`M`是元素文字长度，*`w`*是字符集中最大长度字符所需的字节数。
