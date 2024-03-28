# 8.3.3 创建 SSL 和 RSA 证书和密钥

> 原文：[`dev.mysql.com/doc/refman/8.0/en/creating-ssl-rsa-files.html`](https://dev.mysql.com/doc/refman/8.0/en/creating-ssl-rsa-files.html)

8.3.3.1 使用 MySQL 创建 SSL 和 RSA 证书和密钥

8.3.3.2 使用 openssl 创建 SSL 证书和密钥

8.3.3.3 使用 openssl 创建 RSA 密钥

以下讨论描述了如何在 MySQL 中创建 SSL 和 RSA 支持所需的文件。文件的创建可以使用 MySQL 自身提供的工具，也可以直接调用 **openssl** 命令来执行。

SSL 证书和密钥文件使 MySQL 能够支持使用 SSL 进行加密连接。参见 第 8.3.1 节，“配置 MySQL 使用加密连接”。

RSA 密钥文件使 MySQL 能够支持通过 `sha256_password` 或 `caching_sha2_password` 插件对由未加密连接进行身份验证的帐户进行安全密码交换。参见 第 8.4.1.3 节，“SHA-256 可插拔认证”，以及 第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。
