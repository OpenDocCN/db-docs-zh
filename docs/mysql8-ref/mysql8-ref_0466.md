# 8.6 MySQL 企业加密

> 原文：[`dev.mysql.com/doc/refman/8.0/en/enterprise-encryption.html`](https://dev.mysql.com/doc/refman/8.0/en/enterprise-encryption.html)

8.6.1 MySQL 企业加密安装和升级

8.6.2 配置 MySQL 企业加密

8.6.3 MySQL 企业加密用法和示例

8.6.4 MySQL 企业加密功能参考

8.6.5 MySQL 企业加密组件功能描述

8.6.6 MySQL 企业加密传统功能描述

注意

MySQL 企业加密是 MySQL 企业版中包含的扩展，这是一个商业产品。要了解更多关于商业产品的信息，请访问[`www.mysql.com/products/`](https://www.mysql.com/products/)。

MySQL 企业版包括一组加密函数，这些函数在 SQL 级别暴露了 OpenSSL 的功能。这些函数使企业应用程序能够执行以下操作：

+   使用公钥非对称加密实现额外的数据保护

+   创建公钥、私钥和数字签名

+   执行非对称加密和解密

+   使用加密哈希进行数字签名和数据验证和验证

在 MySQL 8.0.30 之前的版本中，这些功能基于`openssl_udf`共享库。从 MySQL 8.0.30 开始，它们由 MySQL 组件`component_enterprise_encryption`提供。
