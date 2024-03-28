# 8.6.6 MySQL 企业加密传统函数描述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/enterprise-encryption-functions-legacy.html`](https://dev.mysql.com/doc/refman/8.0/en/enterprise-encryption-functions-legacy.html)

在 MySQL 8.0.30 之前的版本中，MySQL 企业加密的函数基于`openssl_udf`共享库。本参考描述了这些函数。如果已安装这些函数，则它们在后续版本中仍然可用，但已被弃用。

有关升级到由 MySQL 组件`component_enterprise_encryption`提供的新组件函数的信息，以及传统函数与组件函数之间的行为差异列表，请参阅升级 MySQL 企业加密。

组件函数的参考资料是第 8.6.5 节，“MySQL 企业加密组件函数描述”。

MySQL 企业加密函数具有以下一般特征：

+   对于错误类型或不正确数量的参数，每个函数都会返回错误。

+   如果参数不适合允许函数执行请求的操作，则它会返回适当的`NULL`或 0。例如，如果函数不支持指定的算法、密钥长度过短或过长，或者期望为 PEM 格式密钥字符串的字符串不是有效密钥。

+   底层 SSL 库负责随机性初始化。

几个传统函数接受加密算法参数。以下表总结了每个函数支持的算法。

**表 8.49 函数支持的算法**

| 函数 | 支持的算法 |
| --- | --- |
| `asymmetric_decrypt()` | RSA |
| `asymmetric_derive()` | DH |
| `asymmetric_encrypt()` | RSA |
| `asymmetric_sign()` | RSA, DSA |
| `asymmetric_verify()` | RSA, DSA |
| `create_asymmetric_priv_key()` | RSA, DSA, DH |
| `create_asymmetric_pub_key()` | RSA, DSA, DH |
| `create_dh_parameters()` | DH |

注意

尽管您可以使用 RSA、DSA 或 DH 加密算法之一创建密钥，但其他接受密钥参数的传统函数可能仅接受特定类型的密钥。例如，`asymmetric_encrypt()`和`asymmetric_decrypt()`仅接受 RSA 密钥。

更多示例和讨论，请参阅第 8.6.3 节，“MySQL 企业加密用法和示例”。

+   `asymmetric_decrypt(*`algorithm`*, *`crypt_str`*, *`key_str`*)`

    使用给定算法和密钥字符串解密加密字符串，并将结果明文作为二进制字符串返回。如果解密失败，则结果为`NULL`。

    `openssl_udf` 共享库函数无法解密由 MySQL 8.0.30 中提供的 `component_enterprise_encryption` 函数生成的内容。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为`'RSA'`。

    *`crypt_str`* 是要解密的加密字符串，该字符串是使用`asymmetric_encrypt()`加密的。

    *`key_str`* 是有效的 PEM 编码 RSA 公钥或私钥。为了成功解密，密钥字符串必须对应于与`asymmetric_encrypt()`一起使用的公钥或私钥字符串，以生成加密字符串。

    有关用法示例，请参阅`asymmetric_encrypt()`的描述。

+   `asymmetric_derive(*`pub_key_str`*, *`priv_key_str`*)`

    使用一方的私钥和另一方的公钥派生对称密钥，并将生成的密钥作为二进制字符串返回。如果密钥派生失败，则结果为`NULL`。

    *`pub_key_str`* 和 *`priv_key_str`* 是使用 DH 算法创建的有效 PEM 编码密钥字符串。

    假设您有两对公钥和私钥：

    ```sql
    SET @dhp = create_dh_parameters(1024);
    SET @priv1 = create_asymmetric_priv_key('DH', @dhp);
    SET @pub1 = create_asymmetric_pub_key('DH', @priv1);
    SET @priv2 = create_asymmetric_priv_key('DH', @dhp);
    SET @pub2 = create_asymmetric_pub_key('DH', @priv2);
    ```

    假设进一步假设您使用一对密钥的私钥和另一对密钥的公钥来创建对称密钥字符串。那么这个对称密钥的身份关系成立：

    ```sql
    asymmetric_derive(@pub1, @priv2) = asymmetric_derive(@pub2, @priv1)
    ```

    此示例需要 DH 私钥/公钥作为输入，使用共享的对称密钥创建。通过将密钥长度传递给`create_dh_parameters()`来创建密钥，然后将密钥作为“密钥长度”传递给`create_asymmetric_priv_key()`。

    ```sql
    -- Generate DH shared symmetric secret
    SET @dhp = create_dh_parameters(1024);
    -- Generate DH key pairs
    SET @algo = 'DH';
    SET @priv1 = create_asymmetric_priv_key(@algo, @dhp);
    SET @pub1 = create_asymmetric_pub_key(@algo, @priv1);
    SET @priv2 = create_asymmetric_priv_key(@algo, @dhp);
    SET @pub2 = create_asymmetric_pub_key(@algo, @priv2);

    -- Generate symmetric key using public key of first party,
    -- private key of second party
    SET @sym1 = asymmetric_derive(@pub1, @priv2);

    -- Or use public key of second party, private key of first party
    SET @sym2 = asymmetric_derive(@pub2, @priv1);
    ```

+   `asymmetric_encrypt(*`algorithm`*, *`str`*, *`key_str`*)`

    使用给定的算法和密钥字符串加密字符串，并将生成的密文作为二进制字符串返回。如果加密失败，则结果为`NULL`。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为`'RSA'`。

    *`str`* 是要加密的字符串。此字符串的长度不能大于密钥字符串长度（以字节为单位），减去 11（用于填充）。

    *`key_str`* 是有效的 PEM 编码 RSA 公钥或私钥。

    要恢复原始未加密的字符串，将加密字符串与用于加密的密钥对的另一部分一起传递给`asymmetric_decrypt()`，如下例所示：

    ```sql
    -- Generate private/public key pair
    SET @priv = create_asymmetric_priv_key('RSA', 1024);
    SET @pub = create_asymmetric_pub_key('RSA', @priv);

    -- Encrypt using private key, decrypt using public key
    SET @ciphertext = asymmetric_encrypt('RSA', 'The quick brown fox', @priv);
    SET @plaintext = asymmetric_decrypt('RSA', @ciphertext, @pub);

    -- Encrypt using public key, decrypt using private key
    SET @ciphertext = asymmetric_encrypt('RSA', 'The quick brown fox', @pub);
    SET @plaintext = asymmetric_decrypt('RSA', @ciphertext, @priv);
    ```

    假设：

    ```sql
    SET @s = a string to be encrypted
    SET @priv = a valid private RSA key string in PEM format
    SET @pub = the corresponding public RSA key string in PEM format
    ```

    那么这些身份关系成立：

    ```sql
    asymmetric_decrypt('RSA', asymmetric_encrypt('RSA', @s, @priv), @pub) = @s
    asymmetric_decrypt('RSA', asymmetric_encrypt('RSA', @s, @pub), @priv) = @s
    ```

+   `asymmetric_sign(*`algorithm`*, *`digest_str`*, *`priv_key_str`*, *`digest_type`*)`

    使用私钥字符串对摘要字符串进行签名，并将签名作为二进制字符串返回。如果签名失败，则结果为`NULL`。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为`'RSA'`和`'DSA'`。

    *`digest_str`* 是摘要字符串。摘要字符串可以通过调用`create_digest()`生成。

    *`priv_key_str`* 是用于签署摘要字符串的私钥字符串。它可以是有效的 PEM 编码的 RSA 私钥或 DSA 私钥。

    *`digest_type`* 是用于签署数据的算法。支持的*`digest_type`*值为`'SHA224'`、`'SHA256'`、`'SHA384'`和`'SHA512'`。

    有关用法示例，请参阅`asymmetric_verify()`的描述。

+   `asymmetric_verify(*`algorithm`*, *`digest_str`*, *`sig_str`*, *`pub_key_str`*, *`digest_type`*)`

    验证签名字符串是否与摘要字符串匹配，并返回 1 或 0 以指示验证成功或失败。如果验证失败，则结果为`NULL`。

    `openssl_udf`共享库函数无法验证由 MySQL 8.0.30 提供的`component_enterprise_encryption`函数生成的内容。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为`'RSA'`和`'DSA'`。

    *`digest_str`* 是摘要字符串。需要一个摘要字符串，可以通过调用`create_digest()`生成。

    *`sig_str`* 是要验证的签名字符串。可以通过调用`asymmetric_sign()`生成签名字符串。

    *`pub_key_str`* 是签名者的公钥字符串。它对应于传递给`asymmetric_sign()`以生成签名字符串的私钥。它必须是有效的 PEM 编码的 RSA 公钥或 DSA 公钥。

    *`digest_type`* 是用于签署数据的算法。支持的*`digest_type`*值为`'SHA224'`、`'SHA256'`、`'SHA384'`和`'SHA512'`。

    ```sql
    -- Set the encryption algorithm and digest type
    SET @algo = 'RSA';
    SET @dig_type = 'SHA224';

    -- Create private/public key pair
    SET @priv = create_asymmetric_priv_key(@algo, 1024);
    SET @pub = create_asymmetric_pub_key(@algo, @priv);

    -- Generate digest from string
    SET @dig = create_digest(@dig_type, 'The quick brown fox');

    -- Generate signature for digest and verify signature against digest
    SET @sig = asymmetric_sign(@algo, @dig, @priv, @dig_type);
    SET @verf = asymmetric_verify(@algo, @dig, @sig, @pub, @dig_type);
    ```

+   `create_asymmetric_priv_key(*`algorithm`*, {*`key_len`*|*`dh_secret`*})`

    使用给定的算法和密钥长度或 DH 密钥创建私钥，并以 PEM 格式的二进制字符串形式返回密钥。密钥采用 PKCS #1 格式。如果密钥生成失败，则结果为`NULL`。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为`'RSA'`、`'DSA'`和`'DH'`。

    *`key_len`* 是 RSA 和 DSA 密钥的位数。如果超过最大允许的密钥长度或指定小于最小值，密钥生成将失败，结果为 null 输出。RSA 算法的最小允许密钥长度为 1,024 位，RSA 算法的最大允许密钥长度为 16,384 位，DSA 算法的最大允许密钥长度为 10,000 位。这些密钥长度限制是 OpenSSL 强加的约束。服务器管理员可以通过设置 `MYSQL_OPENSSL_UDF_RSA_BITS_THRESHOLD`、`MYSQL_OPENSSL_UDF_DSA_BITS_THRESHOLD` 和 `MYSQL_OPENSSL_UDF_DH_BITS_THRESHOLD` 环境变量来对最大密钥长度施加额外限制。参见 Section 8.6.2, “配置 MySQL 企业加密”。

    注意

    生成更长的密钥可能会消耗大量 CPU 资源。通过使用环境变量限制密钥长度，您可以在满足安全需求的同时平衡资源使用。

    *`dh_secret`* 是一个共享的 DH 密钥，必须传递给 DH 密钥而不是密钥长度。要创建这个密钥，请将密钥长度传递给 `create_dh_parameters()`。

    此示例创建了一个 2,048 位的 DSA 私钥，然后从私钥派生出一个公钥：

    ```sql
    SET @priv = create_asymmetric_priv_key('DSA', 2048);
    SET @pub = create_asymmetric_pub_key('DSA', @priv);
    ```

    有关显示 DH 密钥生成的示例，请参阅 `asymmetric_derive()` 的描述。

+   `create_asymmetric_pub_key(*`algorithm`*, *`priv_key_str`*)`

    使用给定算法从给定私钥派生公钥，并以 PEM 格式的二进制字符串返回密钥。密钥采用 PKCS #1 格式。如果密钥派生失败，则结果为 `NULL`。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为 `'RSA'`、`'DSA'` 和 `'DH'`。

    *`priv_key_str`* 是一个有效的 PEM 编码的 RSA、DSA 或 DH 私钥。

    有关用法示例，请参阅 `create_asymmetric_priv_key()` 的描述。

+   `create_dh_parameters(*`key_len`*)`

    创建一个用于生成 DH 私钥/公钥对的共享密钥，并返回一个二进制字符串，可以传递给 `create_asymmetric_priv_key()`。如果密钥生成失败，则结果为 `NULL`。

    *`key_len`* 是密钥长度。位数最小和最大的密钥长度分别为 1,024 和 10,000。这些密钥长度限制是 OpenSSL 强加的约束。服务器管理员可以通过设置 `MYSQL_OPENSSL_UDF_RSA_BITS_THRESHOLD`、`MYSQL_OPENSSL_UDF_DSA_BITS_THRESHOLD` 和 `MYSQL_OPENSSL_UDF_DH_BITS_THRESHOLD` 环境变量来对最大密钥长度施加额外限制。参见 Section 8.6.2, “配置 MySQL 企业加密”。

    有关如何使用返回值生成对称密钥的示例，请参阅`asymmetric_derive()`的描述。

    ```sql
    SET @dhp = create_dh_parameters(1024);
    ```

+   `create_digest(*`digest_type`*, *`str`*)`

    使用给定的字符串和摘要类型创建摘要，并将摘要作为二进制字符串返回。如果摘要生成失败，则结果为`NULL`。

    生成的摘要字符串适用于与`asymmetric_sign()`和`asymmetric_verify()`一起使用。这些函数需要一个摘要。

    *`digest_type`*是用于生成摘要字符串的摘要算法。支持的*`digest_type`*值为`'SHA224'`、`'SHA256'`、`'SHA384'`和`'SHA512'`。

    *`str`*是要生成摘要的非空数据字符串。

    ```sql
    SET @dig = create_digest('SHA512', 'The quick brown fox');
    ```
