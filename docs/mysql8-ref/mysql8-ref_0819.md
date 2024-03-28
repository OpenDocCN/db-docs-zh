# 14.13 加密和压缩函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/encryption-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html)

**表 14.18 加密函数**

| 名称 | 描述 |
| --- | --- |
| `AES_DECRYPT()` | 使用 AES 解密 |
| `AES_ENCRYPT()` | 使用 AES 加密 |
| `COMPRESS()` | 以二进制字符串形式返回结果 |
| `MD5()` | 计算 MD5 校验和 |
| `RANDOM_BYTES()` | 返回一个随机字节向量 |
| `SHA1()`, `SHA()` | 计算 SHA-1 160 位校验和 |
| `SHA2()` | 计算 SHA-2 校验和 |
| `STATEMENT_DIGEST()` | 计算语句摘要哈希值 |
| `STATEMENT_DIGEST_TEXT()` | 计算规范语句摘要 |
| `UNCOMPRESS()` | 解压缩压缩的字符串 |
| `UNCOMPRESSED_LENGTH()` | 返回压缩前字符串的长度 |
| `VALIDATE_PASSWORD_STRENGTH()` | 确定密码强度 |
| 名称 | 描述 |

许多加密和压缩函数返回的字符串可能包含任意字节值。如果您想存储这些结果，请使用具有`VARBINARY`或`BLOB`二进制字符串数据类型的列。这样可以避免由于使用非二进制字符串数据类型（`CHAR`, `VARCHAR`, `TEXT`）可能导致的尾随空格删除或字符集转换等潜在问题，从而改变数据值。

一些加密函数返回 ASCII 字符的字符串：`MD5()`, `SHA()`, `SHA1()`, `SHA2()`, `STATEMENT_DIGEST()`, `STATEMENT_DIGEST_TEXT()`。它们的返回值是一个由`character_set_connection`和`collation_connection`系统变量确定的具有字符集和排序规则的字符串。这是一个非二进制字符串，除非字符集是`binary`。

如果应用程序存储来自诸如`MD5()`或`SHA1()`的函数返回十六进制数字字符串的值，可以通过使用`UNHEX()`将十六进制表示转换为二进制形式并将结果存储在`BINARY(*`N`*)`列中获得更有效的存储和比较。每对十六进制数字在二进制形式中需要一个字节，因此*N*的值取决于十六进制字符串的长度。对于`MD5()`值，*N*为 16，对于`SHA1()`值为 20。对于`SHA2()`，*N*的范围从 28 到 32，具体取决于指定结果所需位长度的参数。

在`CHAR`列中存储十六进制字符串的大小惩罚至少是两倍，如果该值存储在使用`utf8mb4`字符集的列中（其中每个字符使用 4 个字节），则最多是八倍。存储字符串还会导致比较速度变慢，因为数值更大，需要考虑字符集排序规则。

假设一个应用程序将`MD5()`字符串值存储在`CHAR(32)`列中：

```sql
CREATE TABLE md5_tbl (md5_val CHAR(32), ...);
INSERT INTO md5_tbl (md5_val, ...) VALUES(MD5('abcdef'), ...);
```

要将十六进制字符串转换为更紧凑的形式，请修改应用程序以使用`UNHEX()`和`BINARY(16)`如下：

```sql
CREATE TABLE md5_tbl (md5_val BINARY(16), ...);
INSERT INTO md5_tbl (md5_val, ...) VALUES(UNHEX(MD5('abcdef')), ...);
```

应用程序应准备处理散列函数为两个不同输入值产生相同值的极为罕见的情况。使碰撞可检测的一种方法是将哈希列设为主键。

注意

MD5 和 SHA-1 算法的漏洞已经被发现。您可能希望考虑使用本节中描述的其他单向加密函数，例如`SHA2()`。

警告

作为加密函数参数提供的密码或其他敏感值在未使用 SSL 连接的情况下以明文形式发送到 MySQL 服务器。此外，这些值会出现在写入的任何 MySQL 日志中。为了避免这些类型的暴露，应用程序可以在将敏感值发送到服务器之前在客户端端对其进行加密。相同的考虑也适用于加密密钥。为了避免暴露这些，应用程序可以使用存储过程在服务器端对值进行加密和解密。

+   [`AES_DECRYPT(*`crypt_str`*,*`key_str`*[,*`init_vector`*][,*`kdf_name`*][,*`salt`*][,*`info | iterations`*])`](encryption-functions.html#function_aes-decrypt)

    此函数使用官方 AES（高级加密标准）算法解密数据。更多信息，请参阅`AES_ENCRYPT()`的描述。

    使用`AES_DECRYPT()`的语句对基于语句的复制不安全。

+   [`AES_ENCRYPT(*`str`*,*`key_str`*[,*`init_vector`*][,*`kdf_name`*][,*`salt`*][,*`info | iterations`*])`](encryption-functions.html#function_aes-encrypt)

    `AES_ENCRYPT()` 和 `AES_DECRYPT()` 实现使用官方 AES（高级加密标准）算法加密和解密数据，此前被称为“Rijndael”。AES 标准允许使用各种密钥长度。默认情况下，这些函数使用 128 位密钥长度实现 AES。可以使用 196 位或 256 位的密钥长度，如后面所述。密钥长度是性能和安全性之间的权衡。

    `AES_ENCRYPT()` 使用密钥字符串 *`key_str`* 对字符串 *`str`* 进行加密，并返回包含加密输出的二进制字符串。`AES_DECRYPT()` 使用密钥字符串 *`key_str`* 对加密字符串 *`crypt_str`* 进行解密，并以十六进制格式返回原始（二进制）字符串。（要将字符串作为明文获取，将结果转换为 `CHAR`。或者，使用 `--skip-binary-as-hex` 启动**mysql**客户端，以使所有二进制值显示为文本。）如果任一函数参数为 `NULL`，函数将返回 `NULL`。如果`AES_DECRYPT()`检测到无效数据或不正确的填充，它将返回 `NULL`。但是，如果输入数据或密钥无效，`AES_DECRYPT()`可能会返回一个非 `NULL` 值（可能是垃圾）。

    截至 MySQL 8.0.30，这些函数支持使用密钥派生函数（KDF）从传递给 *`key_str`* 的信息创建一个密码强度强的秘密密钥。派生密钥用于加密和解密数据，并保留在 MySQL 服务器实例中，用户无法访问。强烈建议使用 KDF，因为它提供比指定自己预制密钥或通过更简单的方法派生密钥更好的安全性。这些函数支持 HKDF（自 OpenSSL 1.1.0 起可用），您可以指定一个可选的盐和上下文特定信息以包含在密钥材料中，以及 PBKDF2（自 OpenSSL 1.0.2 起可用），您可以指定一个可选的盐并设置用于生成密钥的迭代次数。

    `AES_ENCRYPT()`和`AES_DECRYPT()`允许控制块加密模式。`block_encryption_mode`系统变量控制基于块的加密算法的模式。其默认值为`aes-128-ecb`，表示使用 128 位密钥长度和 ECB 模式进行加密。有关此变量允许的值的描述，请参见第 7.1.8 节，“服务器系统变量”。可选的*`init_vector`*参数用于为需要初始化向量的块加密模式提供初始化向量。

    使用`AES_ENCRYPT()`或`AES_DECRYPT()`的语句对于基于语句的复制是不安全的。

    如果在**mysql**客户端内调用`AES_ENCRYPT()`，二进制字符串将使用十六进制表示，取决于`--binary-as-hex`的值。有关该选项的更多信息，请参见第 6.5.1 节，“mysql — The MySQL Command-Line Client”。

    `AES_ENCRYPT()`和`AES_DECRYPT()`函数的参数如下：

    *`str`*

    用于`AES_ENCRYPT()`加密的字符串，使用密钥字符串*`key_str`*，或者（从 MySQL 8.0.30 开始）使用指定 KDF 派生的密钥。字符串可以是任意长度。根据块为单位的算法（如 AES）的要求，*`str`*会自动添加填充以使其成为块的倍数。此填充会被`AES_DECRYPT()`函数自动移除。

    *`crypt_str`*

    用于`AES_DECRYPT()`解密的加密字符串，使用密钥字符串*`key_str`*，或者（从 MySQL 8.0.30 开始）使用指定 KDF 派生的密钥。字符串可以是任意长度。*`crypt_str`*的长度可以根据原始字符串的长度使用以下公式计算：

    ```sql
    16 * (trunc(*string_length* / 16) + 1)
    ```

    *`key_str`*

    加密密钥，或者作为基础使用密钥派生函数（KDF）派生密钥的输入密钥材料。对于相同的数据实例，使用相同的*`key_str`*值进行使用`AES_ENCRYPT()`进行加密和使用`AES_DECRYPT()`进行解密。

    如果您使用 KDF，可以从 MySQL 8.0.30 开始，*`key_str`*可以是任意信息，如密码或口令。在函数的进一步参数中，您指定 KDF 名称，然后添加进一步选项以根据 KDF 适当增加安全性。

    当您使用 KDF 时，函数会从*`key_str`*中传递的信息以及您在其他参数中提供的盐或其他信息创建一个密码学强大的密钥。派生密钥用于加密和解密数据，并保留在 MySQL 服务器实例中，用户无法访问。强烈建议使用 KDF，因为它提供比指定自己的预制密钥或通过更简单的方法派生密钥更好的安全性。

    如果不使用 KDF，对于 128 位的密钥长度，将密钥传递给*`key_str`*参数的最安全方式是创建一个真正随机的 128 位值并将其作为二进制值传递。例如：

    ```sql
    INSERT INTO t
    VALUES (1,AES_ENCRYPT('text',UNHEX('F3229A0B371ED2D9441B830D21A390C3')));
    ```

    可以使用口令通过哈希口令生成 AES 密钥。例如：

    ```sql
    INSERT INTO t
    VALUES (1,AES_ENCRYPT('text', UNHEX(SHA2('My secret passphrase',512))));
    ```

    如果超过 128 位的最大密钥长度，将返回警告。如果不使用 KDF，请不要直接将密码或口令传递给*`key_str`*，先对其进行哈希处理。本文档的早期版本建议采用前一种方法，但不再建议，因为这里显示的示例更安全。

    *`init_vector`*

    用于需要的块加密模式的初始化向量。`block_encryption_mode`系统变量控制模式。对于相同的数据实例，使用相同的*`init_vector`*值进行`AES_ENCRYPT()`加密和`AES_DECRYPT()`解密。

    注意

    如果使用 KDF，必须为此参数指定一个初始化向量或空字符串，以便访问后续参数以定义 KDF。

    对于需要初始化向量的模式，它必须是 16 字节或更长（超过 16 字节的字节将被忽略）。如果缺少*`init_vector`*，则会发生错误。对于不需要初始化向量的模式，如果指定了*`init_vector`*，则会被忽略并生成警告，除非您使用 KDF。

    `block_encryption_mode`系统变量的默认值为`aes-128-ecb`，或 ECB 模式，不需要初始化向量。允许的替代块加密模式 CBC、CFB1、CFB8、CFB128 和 OFB 都需要初始化向量。

    可以通过调用`RANDOM_BYTES(16)`生成一个用于初始化向量的随机字节字符串。

    *`kdf_name`*

    创建密钥派生函数（KDF）的名称，用于从传递给*KDF*的输入密钥材料中创建密钥，并根据 KDF 的要求提供其他参数。此可选参数从 MySQL 8.0.30 起可用。

    对于相同的数据实例，使用相同的*`kdf_name`*值进行`AES_ENCRYPT()`加密和`AES_DECRYPT()`解密。当指定*`kdf_name`*时，必须指定*`init_vector`*，使用有效的初始化向量或如果加密模式不需要初始化向量则使用空字符串。

    支持以下值：

    `hkdf`

    HKDF，从 OpenSSL 1.1.0 起可用。HKDF 从密钥材料中提取一个伪随机密钥，然后将其扩展为其他密钥。使用 HKDF，您可以指定一个可选的盐（*`salt`*）和上下文特定信息，如应用程序细节（*`info`*）以包含在密钥材料中。

    `pbkdf2_hmac`

    PBKDF2，从 OpenSSL 1.0.2 起可用。PBKDF2 将一个伪随机函数应用于密钥材料，并重复这个过程多次以生成密钥。使用 PBKDF2，您可以指定一个可选的盐（*`salt`*）以包含在密钥材料中，并设置用于生成密钥的迭代次数（*`iterations`*）。

    在此示例中，HKDF 被指定为密钥派生函数，提供了盐和上下文信息。初始化向量的参数被包含在内，但为空字符串：

    ```sql
    SELECT AES_ENCRYPT('mytext','mykeystring', '', 'hkdf', 'salt', 'info');
    ```

    在此示例中，PBKDF2 被指定为密钥派生函数，提供了盐，并且迭代次数是推荐最小值的两倍。

    ```sql
    SELECT AES_ENCRYPT('mytext','mykeystring', '', 'pbkdf2_hmac','salt', '2000');
    ```

    *`salt`*

    传递给密钥派生函数（KDF）的盐。此可选参数从 MySQL 8.0.30 起可用。HKDF 和 PBKDF2 都可以使用盐，建议使用盐以帮助防止基于常见密码字典或彩虹表的攻击。

    盐由随机数据组成，为了安全起见，每次加密操作必须使用不同的盐。可以通过调用`RANDOM_BYTES()`生成一个随机字节字符串作为盐。以下示例生成一个 64 位盐：

    ```sql
    SET @salt = RANDOM_BYTES(8);
    ```

    对于相同的数据实例，使用相同的*`salt`*值进行`AES_ENCRYPT()`加密和`AES_DECRYPT()`解密。盐可以安全地与加密数据一起存储。

    *`info`*

    用于在密钥材料中包含 HKDF 的上下文特定信息，例如有关应用程序的信息。当您将`hkdf`指定为 KDF 名称时，此可选参数从 MySQL 8.0.30 起可用。HKDF 将此信息添加到*`key_str`*中指定的密钥材料和*`salt`*中指定的盐中以生成密钥。

    对于相同的数据实例，使用相同的*`info`*值进行使用`AES_ENCRYPT()`进行加密和使用`AES_DECRYPT()`进行解密。

    *`迭代`*

    生成密钥时 PBKDF2 使用的迭代次数。当您将`pbkdf2_hmac`作为 KDF 名称指定时，此可选参数从 MySQL 8.0.30 开始可用。较高的计数值使得对抗暴力破解攻击更加困难，因为对于攻击者来说，计算成本更高，但对于密钥派生过程也是如此。如果不指定此参数，则默认值为 1000，这是 OpenSSL 标准推荐的最低值。

    对于相同的数据实例，使用相同的*`迭代`*值进行使用`AES_ENCRYPT()`进行加密和使用`AES_DECRYPT()`进行解密。

    ```sql
    mysql> SET block_encryption_mode = 'aes-256-cbc';
    mysql> SET @key_str = SHA2('My secret passphrase',512);
    mysql> SET @init_vector = RANDOM_BYTES(16);
    mysql> SET @crypt_str = AES_ENCRYPT('text',@key_str,@init_vector);
    mysql> SELECT CAST(AES_DECRYPT(@crypt_str,@key_str,@init_vector) AS CHAR);
    +-------------------------------------------------------------+
    | CAST(AES_DECRYPT(@crypt_str,@key_str,@init_vector) AS CHAR) |
    +-------------------------------------------------------------+
    | text                                                        |
    +-------------------------------------------------------------+
    ```

+   `COMPRESS(*`string_to_compress`*)`

    压缩字符串并将结果作为二进制字符串返回。此函数要求 MySQL 已经使用诸如`zlib`之类的压缩库进行编译。否则，返回值始终为`NULL`。如果*`string_to_compress`*为`NULL`，返回值也为`NULL`。压缩后的字符串可以使用`UNCOMPRESS()`进行解压缩。

    ```sql
    mysql> SELECT LENGTH(COMPRESS(REPEAT('a',1000)));
     -> 21
    mysql> SELECT LENGTH(COMPRESS(''));
     -> 0
    mysql> SELECT LENGTH(COMPRESS('a'));
     -> 13
    mysql> SELECT LENGTH(COMPRESS(REPEAT('a',16)));
     -> 15
    ```

    压缩后的字符串内容存储如下：

    +   空字符串存储为空字符串。

    +   非空字符串存储为未压缩字符串的 4 字节长度（低字节在前），后跟压缩字符串。如果字符串以空格结尾，则会添加额外的`.`字符，以避免在将结果存储在`CHAR`或`VARCHAR`列中时出现末尾空格修剪问题。（但是，不建议使用诸如`CHAR`或`VARCHAR`之类的非二进制字符串数据类型来存储压缩字符串，因为可能会发生字符集转换。请改用`VARBINARY`或`BLOB`二进制字符串列。)

    如果从**mysql**客户端中调用`COMPRESS()`，二进制字符串将使用十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参见 Section 6.5.1, “mysql — The MySQL Command-Line Client”。

+   `MD5(*`str`*)`

    为字符串计算 MD5 128 位校验和。返回值是一个包含 32 个十六进制数字的字符串，如果参数为`NULL`则返回`NULL`。返回值可以用作哈希键。有关有效存储哈希值的注意事项，请参阅本节开头的注释。

    返回值是连接字符集中的字符串。

    如果启用了 FIPS 模式，`MD5()`将返回`NULL`。请参阅第 8.8 节，“FIPS 支持”。

    ```sql
    mysql> SELECT MD5('testing');
     -> 'ae2b1fca515949e5d54fb22b8ed95575'
    ```

    这是“RSA 数据安全公司 MD5 消息摘要算法”。

    请参阅本节开头关于 MD5 算法的注意事项。

+   `RANDOM_BYTES(*`len`*)`

    此函数返回使用 SSL 库的随机数生成器生成的*`len`*个随机字节的二进制字符串。允许的*`len`*值范围从 1 到 1024。对于超出该范围的值，将发生错误。如果*`len`*为`NULL`，则返回`NULL`。

    `RANDOM_BYTES()`可用于为`AES_DECRYPT()`和`AES_ENCRYPT()`函数提供初始化向量。在该上下文中使用时，*`len`*必须至少为 16。允许使用更大的值，但超过 16 的字节将被忽略。

    `RANDOM_BYTES()`生成一个随机值，使其结果是不确定的。因此，使用此函数的语句对于基于语句的复制是不安全的。

    如果从**mysql**客户端调用`RANDOM_BYTES()`，则二进制字符串将使用十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参阅第 6.5.1 节，“mysql — MySQL 命令行客户端”。

+   `SHA1(*`str`*)`, `SHA(*`str`*)`

    为字符串计算 SHA-1 160 位校验和，如 RFC 3174（安全哈希算法）中所述。返回值是一个包含 40 个十六进制数字的字符串，如果参数为`NULL`则返回`NULL`。此函数的一个可能用途是作为哈希键。有关有效存储哈希值的注意事项，请参阅本节开头的注释。`SHA()`与`SHA1()`是同义词。

    返回值是连接字符集中的字符串。

    ```sql
    mysql> SELECT SHA1('abc');
     -> 'a9993e364706816aba3e25717850c26c9cd0d89d'
    ```

    `SHA1()`可以被视为`MD5()`的密码学上更安全的等价物。然而，请参阅本节开头关于 MD5 和 SHA-1 算法的注意事项。

+   `SHA2(*`str`*, *`hash_length`*)`

    计算 SHA-2 哈希函数族（SHA-224、SHA-256、SHA-384 和 SHA-512）。第一个参数是要进行哈希处理的明文字符串。第二个参数表示所需结果的位长度，必须为 224、256、384、512 或 0（等同于 256）。如果任一参数为 `NULL` 或哈希长度不是允许的值之一，则返回值为 `NULL`。否则，函数结果是包含所需位数的哈希值。请参阅本节开头关于高效存储哈希值的注意事项。

    返回值是连接字符集中的字符串。

    ```sql
    mysql> SELECT SHA2('abc', 224);
     -> '23097d223405d8228642a477bda255b32aadbce4bda0b3f7e36c9da7'
    ```

    仅当 MySQL 配置了 SSL 支持时，此函数才有效。请参阅 Section 8.3, “Using Encrypted Connections”。

    `SHA2()` 在密码学上比 `MD5()` 或 `SHA1()` 更安全。

+   `STATEMENT_DIGEST(*`statement`*)`

    给定一个作为字符串的 SQL 语句，返回连接字符集中的语句摘要哈希值作为字符串，如果参数为 `NULL` 则返回 `NULL`。相关的 `STATEMENT_DIGEST_TEXT()` 函数返回规范化语句摘要。有关语句摘要的信息，请参阅 Section 29.10, “Performance Schema Statement Digests and Sampling”。

    两个函数都使用 MySQL 解析器来解析语句。如果解析失败，将会出现错误。如果语句以文字字符串形式提供，则错误消息仅包含解析错误。

    `max_digest_length` 系统变量确定这些函数用于计算规范化语句摘要的最大字节数。

    ```sql
    mysql> SET @stmt = 'SELECT * FROM mytable WHERE cola = 10 AND colb = 20';
    mysql> SELECT STATEMENT_DIGEST(@stmt);
    +------------------------------------------------------------------+
    | STATEMENT_DIGEST(@stmt)                                          |
    +------------------------------------------------------------------+
    | 3bb95eeade896657c4526e74ff2a2862039d0a0fe8a9e7155b5fe492cbd78387 |
    +------------------------------------------------------------------+
    mysql> SELECT STATEMENT_DIGEST_TEXT(@stmt);
    +----------------------------------------------------------+
    | STATEMENT_DIGEST_TEXT(@stmt)                             |
    +----------------------------------------------------------+
    | SELECT * FROM `mytable` WHERE `cola` = ? AND `colb` = ?  |
    +----------------------------------------------------------+
    ```

+   `STATEMENT_DIGEST_TEXT(*`statement`*)`

    给定一个作为字符串的 SQL 语句，返回连接字符集中的规范化语句摘要作为字符串，如果参数为 `NULL` 则返回 `NULL`。有关更多讨论和示例，请参阅相关 `STATEMENT_DIGEST()` 函数的描述。

+   `UNCOMPRESS(*`string_to_uncompress`*)`

    解压由`COMPRESS()`函数压缩的字符串。如果参数不是压缩值，则结果为`NULL`；如果*`string_to_uncompress`*为`NULL`，则结果也为`NULL`。此函数要求 MySQL 已经使用诸如`zlib`之类的压缩库进行编译。否则，返回值始终为`NULL`。

    ```sql
    mysql> SELECT UNCOMPRESS(COMPRESS('any string'));
     -> 'any string'
    mysql> SELECT UNCOMPRESS('any string');
     -> NULL
    ```

+   `UNCOMPRESSED_LENGTH(*`compressed_string`*)`

    返回压缩前的字符串长度。如果*`compressed_string`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT UNCOMPRESSED_LENGTH(COMPRESS(REPEAT('a',30)));
     -> 30
    ```

+   `VALIDATE_PASSWORD_STRENGTH(*`str`*)`

    给定表示明文密码的参数，此函数返回一个整数，指示密码的强度，如果参数为`NULL`，则返回`NULL`。返回值范围从 0（弱）到 100（强）。

    由`VALIDATE_PASSWORD_STRENGTH()`进行密码评估，由`validate_password`组件执行。如果未安装该组件，函数始终返回 0。有关安装`validate_password`的信息，请参见第 8.4.3 节，“密码验证组件”。要检查或配置影响密码测试的参数，请查看或设置`validate_password`实现的系统变量。请参见第 8.4.3.2 节，“密码验证选项和变量”。

    密码将经过越来越严格的测试，返回值反映了满足的测试，如下表所示。此外，如果启用了`validate_password.check_user_name`系统变量，并且密码与用户名匹配，`VALIDATE_PASSWORD_STRENGTH()`无论如何返回 0，不管其他`validate_password`系统变量如何设置。

    | 密码测试 | 返回值 |
    | --- | --- |
    | 长度 < 4 | 0 |
    | 长度 ≥ 4 且 < `validate_password.length` | 25 |
    | 符合策略 1（`LOW`） | 50 |
    | 符合策略 2（`MEDIUM`） | 75 |
    | 符合策略 3（`STRONG`） | 100 |
