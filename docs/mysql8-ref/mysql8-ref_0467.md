# 8.6.1 MySQL Enterprise Encryption 安装和升级

> 原文：[`dev.mysql.com/doc/refman/8.0/en/enterprise-encryption-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/enterprise-encryption-installation.html)

在 MySQL 8.0.30 之前的版本中，MySQL Enterprise Encryption 提供的函数是通过单独创建它们而安装的，基于`openssl_udf`共享库。从 MySQL 8.0.30 开始，这些函数由 MySQL 组件`component_enterprise_encryption`提供，安装组件会安装所有函数。从该版本开始，`openssl_udf`共享库中的函数已被弃用，您应该升级到组件。

+   从 MySQL 8.0.30 安装

+   到 MySQL 8.0.29 安装

+   升级 MySQL Enterprise Encryption

#### 从 MySQL 8.0.30 安装

从 MySQL 8.0.30 开始，MySQL Enterprise Encryption 的函数由 MySQL 组件`component_enterprise_encryption`提供，而不是从`openssl_udf`共享库安装。如果您正在从早期版本升级到 MySQL 8.0.30，其中使用了 MySQL Enterprise Encryption，您创建的函数仍然可用并受支持。但是，这些旧版函数从此版本开始已被弃用，建议您安装组件。组件函数向后兼容。有关升级信息，请参见升级 MySQL Enterprise Encryption。

如果您正在升级，在安装组件之前，请使用`DROP FUNCTION`语句卸载旧版函数：

```sql
DROP FUNCTION asymmetric_decrypt;
DROP FUNCTION asymmetric_derive;
DROP FUNCTION asymmetric_encrypt;
DROP FUNCTION asymmetric_sign;
DROP FUNCTION asymmetric_verify;
DROP FUNCTION create_asymmetric_priv_key;
DROP FUNCTION create_asymmetric_pub_key;
DROP FUNCTION create_dh_parameters;
DROP FUNCTION create_digest;
```

函数名称必须以小写指定。这些语句需要对`mysql`数据库的`DROP`权限。

要安装组件，请发出一个`INSTALL COMPONENT`语句：

```sql
INSTALL COMPONENT "file://component_enterprise_encryption";
```

`INSTALL COMPONENT`需要`mysql.component`系统表的`INSERT`权限，因为它会向该表添加一行以注册组件。要验证组件是否已安装，请发出：

```sql
SELECT * FROM mysql.component;
```

在`mysql.component`中列出的组件在启动序列期间由加载器服务加载。

如果您需要卸载组件，请发出一个`UNINSTALL COMPONENT`语句：

```sql
UNINSTALL COMPONENT "file://component_enterprise_encryption";
```

查看更多详细信息，请参见第 7.5.1 节，“安装和卸载组件”。

安装组件会安装所有函数，因此您无需像在 MySQL 8.0.30 之前那样使用`CREATE FUNCTION`语句创建它们。卸载组件会卸载所有函数。

安装组件后，如果您希望组件函数支持解密和验证在 MySQL 8.0.30 之前生成的遗留函数产生的内容，请将组件的系统变量`enterprise_encryption.rsa_support_legacy_padding`设置为`ON`。此外，如果您希望更改组件函数生成的 RSA 密钥的最大长度允许值，请使用组件的系统变量`enterprise_encryption.maximum_rsa_key_size`设置一个适当的最大值。有关配置信息，请参见第 8.6.2 节，“配置 MySQL 企业加密”。

#### 安装到 MySQL 8.0.29

在 MySQL 8.0.29 之前，MySQL 企业加密函数位于安装在插件目录（由`plugin_dir`系统变量命名的目录）中的可加载函数库文件中。函数库基本名称为`openssl_udf`，后缀取决于平台。例如，在 Linux 或 Windows 上的文件名分别为`openssl_udf.so`或`openssl_udf.dll`。

要从`openssl_udf`共享库文件安装函数，请使用`CREATE FUNCTION`语句。要从库中加载所有函数，请使用以下一组语句，根据需要调整文件名后缀：

```sql
CREATE FUNCTION asymmetric_decrypt RETURNS STRING
  SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_derive RETURNS STRING
  SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_encrypt RETURNS STRING
  SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_sign RETURNS STRING
  SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_verify RETURNS INTEGER
  SONAME 'openssl_udf.so';
CREATE FUNCTION create_asymmetric_priv_key RETURNS STRING
  SONAME 'openssl_udf.so';
CREATE FUNCTION create_asymmetric_pub_key RETURNS STRING
  SONAME 'openssl_udf.so';
CREATE FUNCTION create_dh_parameters RETURNS STRING
  SONAME 'openssl_udf.so';
CREATE FUNCTION create_digest RETURNS STRING
  SONAME 'openssl_udf.so';
```

安装后，函数将在服务器重新启动时保持安装状态。如果需要卸载函数，请使用`DROP FUNCTION`语句：

```sql
DROP FUNCTION asymmetric_decrypt;
DROP FUNCTION asymmetric_derive;
DROP FUNCTION asymmetric_encrypt;
DROP FUNCTION asymmetric_sign;
DROP FUNCTION asymmetric_verify;
DROP FUNCTION create_asymmetric_priv_key;
DROP FUNCTION create_asymmetric_pub_key;
DROP FUNCTION create_dh_parameters;
DROP FUNCTION create_digest;
```

在`CREATE FUNCTION`和`DROP FUNCTION`语句中，函数名称必须以小写指定。这与在函数调用时使用的方式不同，对于函数调用时，您可以使用任何大小写。

`CREATE FUNCTION`和`DROP FUNCTION`语句分别需要`mysql`数据库的`INSERT`和`DROP`权限。

`openssl_udf`共享库提供的函数允许最小密钥大小为 1024 位。您可以使用`MYSQL_OPENSSL_UDF_RSA_BITS_THRESHOLD`、`MYSQL_OPENSSL_UDF_DSA_BITS_THRESHOLD`和`MYSQL_OPENSSL_UDF_DH_BITS_THRESHOLD`环境变量设置最大密钥大小，如第 8.6.2 节，“配置 MySQL 企业加密”中所述。如果不设置最大密钥大小，RSA 算法的上限为 16384，DSA 算法的上限为 10000，由 OpenSSL 指定。

#### 升级 MySQL 企业加密

如果您从使用`openssl_udf`共享库提供的函数的较早版本升级到 MySQL 8.0.30 或更高版本，则您创建的函数仍然可用并受支持。但是，这些传统函数从 MySQL 8.0.30 开始已被弃用，建议您安装 MySQL 企业加密组件`component_enterprise_encryption`。

在升级之前，在安装组件之前，您必须使用`DROP FUNCTION`语句卸载传统函数。有关如何执行此操作的说明，请参见从 MySQL 8.0.30 安装。

组件函数向后兼容：

+   RSA 公钥和私钥由传统函数生成，可以与组件函数一起使用。

+   使用传统函数加密的数据可以由组件函数解密。

+   由传统函数创建的签名可以使用组件函数进行验证。

为了使组件函数支持由传统函数生成的内容的解密和验证，您必须将系统变量`enterprise_encryption.rsa_support_legacy_padding`设置为`ON`（默认为`OFF`）。有关配置信息，请参见第 8.6.2 节，“配置 MySQL 企业加密”。

由于组件函数使用的填充和密钥格式与当前标准不同，传统函数无法处理由组件函数创建的加密数据、公钥和签名。

`component_enterprise_encryption` 组件提供的新函数在行为和支持方面与 `openssl_udf` 共享库提供的旧版函数有一些差异。其中最重要的是：

+   旧版函数支持较旧的 DSA 算法和 Diffie-Hellman 密钥交换方法。组件函数仅使用普遍首选的 RSA 算法。

+   对于旧版函数，最小 RSA 密钥大小低于当前最佳实践。组件函数遵循当前最佳实践的最小 RSA 密钥大小。

+   旧版函数仅支持 SHA2 用于摘要，并要求摘要用于签名。组件函数还支持 SHA3 用于摘要（前提是使用 OpenSSL 1.1.1），并且不要求摘要用于签名，尽管它们支持。

+   `asymmetric_encrypt()` 旧版函数支持使用私钥进行加密。`asymmetric_encrypt()` 组件函数仅接受公钥。建议您在旧版函数中也仅使用公钥进行加密。

+   `create_dh_parameters()` 和 `asymmetric_derive()` 用于 Diffie-Hellman 密钥交换方法的旧版函数不在 `component_enterprise_encryption` 组件中提供。

表 1 总结了由 `openssl_udf` 共享库提供的旧版函数与 MySQL 8.0.30 中 `component_enterprise_encryption` 组件提供的函数之间在支持和操作方面的技术差异。

**表 8.48 MySQL 企业加密函数**

| 能力 | 旧版函数（到 MySQL 8.0.29） | 组件函数（从 MySQL 8.0.30） |
| --- | --- | --- |
| 加密方法 | RSA, DSA, Diffie-Hellman (DH) | 仅限 RSA |
| 用于加密的密钥 | 私钥或公钥 | 仅限公钥 |
| RSA 密钥格式 | PKCS #1 v1.5 | PKCS #8 |
| 最小 RSA 密钥大小 | 1024 位 | 2048 位 |
| 最大 RSA 密钥大小限制 | 使用环境变量 `MYSQL_OPENSSL_UDF_RSA_BITS_THRESHOLD` 设置，默认限制为算法最大 16384 | 使用系统变量 `enterprise_encryption.maximum_rsa_key_size` 设置，默认限制为 4096 |
| 摘要算法 | SHA2 | SHA2, SHA3 (需要使用 OpenSSL 1.1.1) |
| 签名 | 需要摘要 | 支持摘要但不需要，可以使用任意长度的任意字符串 |
| 输出填充 | RSAES-PKCS1-v1_5 | RSAES-OAEP |
| 签名填充 | RSASSA-PKCS1-v1_5 | RSASSA-PSS |
