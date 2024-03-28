# 8.6.2 配置 MySQL 企业加密

> 原文：[`dev.mysql.com/doc/refman/8.0/en/enterprise-encryption-configuring.html`](https://dev.mysql.com/doc/refman/8.0/en/enterprise-encryption-configuring.html)

MySQL 企业加密允许您限制密钥的长度，以满足您的安全要求，同时平衡资源使用。您还可以配置 MySQL 8.0.30 提供的`component_enterprise_encryption`组件的函数，以支持解密和验证由遗留`openssl_udf`共享库函数生成的内容。

#### 组件函数对遗留函数的解密支持

默认情况下，MySQL 8.0.30 提供的`component_enterprise_encryption`组件的函数不会解密由早期版本中`openssl_udf`共享库提供的遗留函数生成的加密文本，或验证签名。组件函数假定加密文本使用 RSAES-OAEP 填充方案，签名使用 RSASSA-PSS 签名方案。然而，遗留函数生成的加密文本使用 RSAES-PKCS1-v1_5 填充方案，遗留函数生成的签名使用 RSASSA-PKCS1-v1_5 签名方案。

如果您希望组件函数支持 MySQL 8.0.30 之前的遗留函数生成的内容，请将组件的系统变量`enterprise_encryption.rsa_support_legacy_padding`设置为`ON`。该系统变量在安装组件时可用。当您将其设置为`ON`时，组件函数首先尝试解密或验证内容，假设其具有其正常方案。如果这种方式不起作用，它们还会尝试解密或验证内容，假设其具有遗留函数使用的方案。这种行为不是默认设置，因为它会增加处理无法解密或验证的内容所需的时间。如果您不处理由遗留函数生成的内容，请将系统变量保持默认设置为`OFF`。

#### 密钥长度限制

随着密钥长度的增加，MySQL 企业加密的密钥生成函数所需的 CPU 资源量也会增加。对于一些安装来说，如果应用程序频繁生成过长的密钥，这可能导致不可接受的 CPU 使用率。

OpenSSL 指定所有密钥的最小长度为 1024 位。OpenSSL 还指定 RSA 密钥的最大长度为 16384 位，DSA 密钥为 10000 位，DH 密钥为 10000 位。

从 MySQL 8.0.30 开始，`component_enterprise_encryption` 组件提供的函数对于 RSA 密钥有更高的最小长度要求，为 2048 位，这符合当前最佳实践的最小密钥长度。组件的系统变量 `enterprise_encryption.maximum_rsa_key_size` 指定了最大密钥长度，默认为 4096 位。您可以更改此值以允许 OpenSSL 允许的最大长度，即 16384 位。

对于 MySQL 8.0.30 之前的版本，`openssl_udf` 共享库提供的传统函数默认使用 OpenSSL 的最小和最大限制。如果最大值过高，您可以使用以下系统变量指定较低的最大密钥长度：

+   `MYSQL_OPENSSL_UDF_DSA_BITS_THRESHOLD`：`create_asymmetric_priv_key()` 的最大 DSA 密钥长度（以位为单位）。此变量的最小值和最大值分别为 1024 和 10000。

+   `MYSQL_OPENSSL_UDF_RSA_BITS_THRESHOLD`：`create_asymmetric_priv_key()` 的最大 RSA 密钥长度（以位为单位）。此变量的最小值和最大值分别为 1024 和 16384。

+   `MYSQL_OPENSSL_UDF_DH_BITS_THRESHOLD`：`create_dh_parameters()` 的最大密钥长度（以位为单位）。此变量的最小值和最大值分别为 1024 和 10000。

要使用这些环境变量中的任何一个，请在启动服务器的进程的环境中设置它们。如果设置了这些变量，它们的值将优先于 OpenSSL 强制的最大密钥长度。例如，要为 `create_asymmetric_priv_key()` 的 DSA 和 RSA 密钥设置最大密钥长度为 4096 位，请设置这些变量：

```sql
export MYSQL_OPENSSL_UDF_DSA_BITS_THRESHOLD=4096
export MYSQL_OPENSSL_UDF_RSA_BITS_THRESHOLD=4096
```

本示例使用 Bourne shell 语法。其他 shell 的语法可能有所不同。
