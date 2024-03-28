# 8.6.3 MySQL 企业加密用法和示例

> 原文：[`dev.mysql.com/doc/refman/8.0/en/enterprise-encryption-usage.html`](https://dev.mysql.com/doc/refman/8.0/en/enterprise-encryption-usage.html)

在应用程序中使用 MySQL 企业加密时，调用适合所需操作的函数。本节演示了如何执行一些代表性任务。

在 MySQL 8.0.30 之前的版本中，MySQL 企业加密的函数基于`openssl_udf`共享库。从 MySQL 8.0.30 开始，这些函数由 MySQL 组件`component_enterprise_encryption`提供。在某些情况下，组件函数的行为与`openssl_udf`提供的旧版函数的行为不同。有关差异的列表，请参见升级 MySQL 企业加密。有关每个组件函数行为的详细信息，请参见第 8.6.4 节，“MySQL 企业加密函数参考”。

如果安装了旧版函数，然后升级到 MySQL 8.0.30 或更高版本，那么您创建的函数仍然可用，受支持，并且继续以相同方式工作。但是，从 MySQL 8.0.30 开始，它们已被弃用，并建议安装 MySQL 企业加密组件`component_enterprise_encryption`。有关升级说明，请参见从 MySQL 8.0.30 安装。

选择密钥长度和加密算法时，需要考虑以下一般因素：

+   私钥和公钥的加密强度随着密钥大小的增加而增加，但密钥生成的时间也会增加。

+   对于旧版函数，生成 DH 密钥比 RSA 或 DSA 密钥花费的时间更长。从 MySQL 8.0.30 开始，组件函数仅支持 RSA 密钥。

+   非对称加密函数消耗的资源比对称函数多。它们适用于加密少量数据以及创建和验证签名。对于加密大量数据，对称加密函数更快。MySQL 服务器提供了`AES_ENCRYPT()`和`AES_DECRYPT()`函数用于对称加密。

可以在运行时创建密钥字符串值，并使用`SET`、`SELECT`或`INSERT`将其存储到变量或表中。此示例适用于组件函数和旧版函数。

```sql
SET @priv1 = create_asymmetric_priv_key('RSA', 2048);
SELECT create_asymmetric_priv_key('RSA', 2048) INTO @priv2;
INSERT INTO t (key_col) VALUES(create_asymmetric_priv_key('RSA', 1024));
```

存储在文件中的密钥字符串值可以由具有`FILE`权限的用户使用`LOAD_FILE()`函数读取。摘要和签名字符串可以类似地处理。

+   创建私钥/公钥对

+   使用公钥加密数据和私钥解密数据

+   从字符串生成摘要

+   使用密钥对生成摘要

#### 创建私钥/公钥对

这个示例适用于组件功能和传统功能：

```sql
-- Encryption algorithm
SET @algo = 'RSA';
-- Key length in bits; make larger for stronger keys
SET @key_len = 2048;

-- Create private key
SET @priv = create_asymmetric_priv_key(@algo, @key_len);
-- Derive corresponding public key from private key, using same algorithm
SET @pub = create_asymmetric_pub_key(@algo, @priv);
```

您可以使用密钥对来加密和解密数据，或者用来签名和验证数据。

#### 使用公钥加密数据和私钥解密数据

这个示例适用于组件功能和传统功能。在这两种情况下，密钥对的成员必须是 RSA 密钥：

```sql
SET @ciphertext = asymmetric_encrypt(@algo, 'My secret text', @pub);
SET @plaintext = asymmetric_decrypt(@algo, @ciphertext, @priv);
```

#### 从字符串生成摘要

这个示例适用于组件功能和传统功能：

```sql
-- Digest type
SET @dig_type = 'SHA512';

-- Generate digest string
SET @dig = create_digest(@dig_type, 'My text to digest');
```

#### 使用密钥对生成摘要

密钥对可用于签署数据，然后验证签名是否与摘要匹配。这个示例适用于组件功能和传统功能：

```sql
-- Encryption algorithm; keys must
-- have been created using same algorithm
SET @algo = 'RSA';
–- Digest algorithm to sign the data
SET @dig_type = 'SHA512';

-- Generate signature for digest and verify signature against digest
SET @sig = asymmetric_sign(@algo, @dig, @priv, @dig_type);
-- Verify signature against digest
SET @verf = asymmetric_verify(@algo, @dig, @sig, @pub, @dig_type);
```

对于传统功能，签名需要一个摘要。对于组件功能，签名不需要摘要，并且可以使用任何数据字符串。这些功能中的摘要类型指的是用于签署数据的算法，而不是用于创建签名的原始输入的算法。这个示例是针对组件功能的：

```sql
-- Encryption algorithm; keys must
-- have been created using same algorithm
SET @algo = 'RSA';
–- Arbitrary text string for signature
SET @text = repeat('j', 256);
–- Digest algorithm to sign the data
SET @dig_type = 'SHA512';

-- Generate signature for digest and verify signature against digest
SET @sig = asymmetric_sign(@algo, @text, @priv, @dig_type);
-- Verify signature against digest
SET @verf = asymmetric_verify(@algo, @text, @sig, @pub, @dig_type);
```
