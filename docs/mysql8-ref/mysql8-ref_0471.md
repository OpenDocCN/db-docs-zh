# 8.6.5 MySQL Enterprise Encryption 组件功能描述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/enterprise-encryption-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/enterprise-encryption-functions.html)

从 MySQL 8.0.30 版本开始，MySQL Enterprise Encryption 的功能由 MySQL 组件 `component_enterprise_encryption` 提供。本参考描述了这些功能。

有关升级到由 MySQL 组件 `component_enterprise_encryption` 提供的新组件功能以及传统功能与组件功能之间行为差异的列表，请参阅 升级 MySQL Enterprise Encryption。

在 MySQL 8.0.30 之前基于 `openssl_udf` 共享库的传统功能的参考是 第 8.6.6 节，“MySQL Enterprise Encryption 传统功能描述”。

MySQL Enterprise Encryption 功能具有以下一般特性：

+   对于错误类型的参数或不正确数量的参数，每个函数都会返回错误。

+   如果参数不适合允许函数执行请求的操作，则返回适当的 `NULL` 或 0。例如，如果函数不支持指定的算法，密钥长度过短或过长，或者预期为 PEM 格式密钥字符串的字符串不是有效密钥。

+   底层 SSL 库负责随机性初始化。

组件功能仅支持 RSA 加密算法。

更多示例和讨论，请参阅 第 8.6.3 节，“MySQL Enterprise Encryption 使用和示例”。

+   `asymmetric_decrypt(*`algorithm`*, *`data_str`*, *`priv_key_str`*)`

    使用给定算法和密钥字符串解密加密字符串，并将结果明文作为二进制字符串返回。如果解密失败，则结果为 `NULL`。

    对于在 MySQL 8.0.29 之前使用的此函数的传统版本，请参阅 第 8.6.6 节，“MySQL Enterprise Encryption 传统功能描述”。

    默认情况下，`component_enterprise_encryption` 函数假定加密文本使用 RSAES-OAEP 填充方案。如果系统变量 `enterprise_encryption.rsa_support_legacy_padding` 设置为 `ON`（默认为 `OFF`），则该函数支持通过遗留的 `openssl_udf` 共享库函数加密的内容的解密。当设置为 `ON` 时，该函数还支持 RSAES-PKCS1-v1_5 填充方案，这是遗留的 `openssl_udf` 共享库函数使用的填充方案。当设置为 `OFF` 时，无法解密由遗留函数加密的内容，对于这样的内容，函数返回空输出。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为 `'RSA'`。

    *`data_str`* 是要解密的加密字符串，该字符串是使用 `asymmetric_encrypt()` 加密的。

    *`priv_key_str`* 是一个有效的 PEM 编码的 RSA 私钥。为了成功解密，密钥字符串必须对应于与 `asymmetric_encrypt()` 一起使用的公钥字符串，以产生加密字符串。`asymmetric_encrypt()` 组件函数仅支持使用公钥进行加密，因此解密需要使用相应的私钥。

    有关用法示例，请参阅 `asymmetric_encrypt()` 的描述。

+   `asymmetric_encrypt(*`algorithm`*, *`data_str`*, *`pub_key_str`*)`

    使用给定的算法和密钥字符串加密字符串，并将结果密文作为二进制字符串返回。如果加密失败，则结果为 `NULL`。

    对于 MySQL 8.0.29 之前使用的旧版本函数，请参阅 Section 8.6.6, “MySQL Enterprise Encryption Legacy Function Descriptions”。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为 `'RSA'`。

    *`data_str`* 是要加密的字符串。此字符串的长度不能大于字节中的密钥字符串长度减去 42（用于填充）。

    *`pub_key_str`* 是一个有效的 PEM 编码的 RSA 公钥。`asymmetric_encrypt()` 组件函数仅支持使用公钥进行加密。

    要恢复原始未加密字符串，请将加密字符串传递给 `asymmetric_decrypt()`，以及用于加密的密钥对的另一部分，如下例所示：

    ```sql
    -- Generate private/public key pair
    SET @priv = create_asymmetric_priv_key('RSA', 2048);
    SET @pub = create_asymmetric_pub_key('RSA', @priv);

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

    然后这些身份关系成立：

    ```sql
    asymmetric_decrypt('RSA', asymmetric_encrypt('RSA', @s, @pub), @priv) = @s
    ```

+   `asymmetric_sign(*`algorithm`*, *`text`*, *`priv_key_str`*, *`digest_type`*)`

    使用私钥签署摘要字符串或数据字符串，并将签名作为二进制字符串返回。如果签名失败，则结果为 `NULL`。

    对于在 MySQL 8.0.29 之前使用的此函数的旧版本，请参见 第 8.6.6 节，“MySQL 企业加密遗留函数描述”。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为 `'RSA'`。

    *`text`* 是数据字符串或摘要字符串。该函数接受摘要但不要求摘要，因为它还能处理任意长度的数据字符串。摘要字符串可以通过调用 `create_digest()` 生成。

    *`priv_key_str`* 是用于签署摘要字符串的私钥字符串。它必须是有效的 PEM 编码的 RSA 私钥。

    *`digest_type`* 是用于签署数据的算法。当使用 OpenSSL 1.0.1 时，支持的 *`digest_type`* 值为 `'SHA224'`、`'SHA256'`、`'SHA384'` 和 `'SHA512'`。如果使用 OpenSSL 1.1.1，则还可用额外的 *`digest_type`* 值 `'SHA3-224'`、`'SHA3-256'`、`'SHA3-384'` 和 `'SHA3-512'`。

    有关用法示例，请参阅 `asymmetric_verify()` 的描述。

+   `asymmetric_verify(*`algorithm`*, *`text`*, *`sig_str`*, *`pub_key_str`*, *`digest_type`*)`

    验证签名字符串是否与摘要字符串匹配，并返回 1 或 0 表示验证成功或失败。如果验证失败，则结果为 `NULL`。

    对于在 MySQL 8.0.29 之前使用的此函数的旧版本，请参见 第 8.6.6 节，“MySQL 企业加密遗留函数描述”。

    默认情况下，`component_enterprise_encryption` 函数假定签名使用 RSASSA-PSS 签名方案。如果系统变量 `enterprise_encryption.rsa_support_legacy_padding` 设置为 `ON`（默认为 `OFF`），该函数支持验证由旧版 `openssl_udf` 共享库函数生成的签名。当设置为 `ON` 时，该函数还支持 RSASSA-PKCS1-v1_5 签名方案，如旧版 `openssl_udf` 共享库函数所使用的。当设置为 `OFF` 时，无法验证由旧版函数生成的签名，并且对于这种内容，函数返回空输出。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为 `'RSA'`。

    *`text`* 是数据字符串或摘要字符串。该组件函数接受摘要但不需要它们，因为它还能处理任意长度的数据字符串。可以通过调用 `create_digest()` 来生成摘要字符串。

    *`sig_str`* 是要验证的签名字符串。可以通过调用 `asymmetric_sign()` 来生成签名字符串。

    *`pub_key_str`* 是签名者的公钥字符串。它对应于传递给 `asymmetric_sign()` 以生成签名字符串的私钥。它必须是有效的 PEM 编码的 RSA 公钥。

    *`digest_type`* 是用于签署数据的算法。在使用 OpenSSL 1.0.1 时，支持的 *`digest_type`* 值为 `'SHA224'`、`'SHA256'`、`'SHA384'` 和 `'SHA512'`。如果使用 OpenSSL 1.1.1，则还可以使用额外的 *`digest_type`* 值 `'SHA3-224'`、`'SHA3-256'`、`'SHA3-384'` 和 `'SHA3-512'`。

    ```sql
    -- Set the encryption algorithm and digest type
    SET @algo = 'RSA';
    SET @dig_type = 'SHA512';

    -- Create private/public key pair
    SET @priv = create_asymmetric_priv_key(@algo, 2048);
    SET @pub = create_asymmetric_pub_key(@algo, @priv);

    -- Generate digest from string
    SET @dig = create_digest(@dig_type, 'The quick brown fox');

    -- Generate signature for digest and verify signature against digest
    SET @sig = asymmetric_sign(@algo, @dig, @priv, @dig_type);
    SET @verf = asymmetric_verify(@algo, @dig, @sig, @pub, @dig_type);
    ```

+   `create_asymmetric_priv_key(*`algorithm`*, *`key_length`*)`

    使用给定的算法和密钥长度创建私钥，并以 PEM 格式的二进制字符串形式返回密钥。密钥采用 PKCS #8 格式。如果密钥生成失败，则结果为 `NULL`。

    对于 MySQL 8.0.29 之前使用的旧版本此函数，请参阅 Section 8.6.6, “MySQL Enterprise Encryption Legacy Function Descriptions”。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为 `'RSA'`。

    *`key_length`* 是密钥长度（以位为单位）。如果超过最大允许的密钥长度或指定少于最小值，则密钥生成失败，结果为 null 输出。密钥长度的最小允许值为 2048 位。最大允许的密钥长度是 `enterprise_encryption.maximum_rsa_key_size` 系统变量的值，默认为 4096。最大设置为 16384，这是 RSA 算法允许的最大密钥长度。请参阅 Section 8.6.2, “Configuring MySQL Enterprise Encryption”。

    注意

    生成更长的密钥可能会消耗大量 CPU 资源。使用 `enterprise_encryption.maximum_rsa_key_size` 系统变量限制密钥长度，可以在提供足够安全性的同时平衡资源使用。

    此示例创建一个 2048 位的 RSA 私钥，然后从私钥派生公钥：

    ```sql
    SET @priv = create_asymmetric_priv_key('RSA', 2048);
    SET @pub = create_asymmetric_pub_key('RSA', @priv);
    ```

+   `create_asymmetric_pub_key(*`algorithm`*, *`priv_key_str`*)`

    使用给定算法从给定私钥派生公钥，并以 PEM 格式的二进制字符串返回该密钥。密钥采用 PKCS #8 格式。如果密钥派生失败，则结果为 `NULL`。

    对于 MySQL 8.0.29 之前使用的旧版函数，请参阅 Section 8.6.6, “MySQL Enterprise Encryption Legacy Function Descriptions”。

    *`algorithm`* 是用于创建密钥的加密算法。支持的算法值为 `'RSA'`。

    *`priv_key_str`* 是有效的 PEM 编码的 RSA 私钥。

    有关用法示例，请参阅 `create_asymmetric_priv_key()` 的描述。

+   `create_digest(*`digest_type`*, *`str`*)`

    使用给定摘要类型从给定字符串创建摘要，并将摘要作为二进制字符串返回。如果摘要生成失败，则结果为 `NULL`。

    对于 MySQL 8.0.29 之前使用的旧版函数，请参阅 Section 8.6.6, “MySQL Enterprise Encryption Legacy Function Descriptions”。

    生成的摘要字符串适用于与 `asymmetric_sign()` 和 `asymmetric_verify()` 一起使用。这些函数的组件版本接受摘要但不需要它们，因为它们能够处理任意长度的数据。

    *`digest_type`* 是用于生成摘要字符串的摘要算法。当使用 OpenSSL 1.0.1 时，支持的 *`digest_type`* 值为 `'SHA224'`、`'SHA256'`、`'SHA384'` 和 `'SHA512'`。如果使用 OpenSSL 1.1.1，则还可使用额外的 *`digest_type`* 值 `'SHA3-224'`、`'SHA3-256'`、`'SHA3-384'` 和 `'SHA3-512'`。

    *`str`* 是要生成摘要的非空数据字符串。

    ```sql
    SET @dig = create_digest('SHA512', 'The quick brown fox');
    ```
