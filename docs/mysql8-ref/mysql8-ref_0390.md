> 原文：[`dev.mysql.com/doc/refman/8.0/en/creating-rsa-files-using-openssl.html`](https://dev.mysql.com/doc/refman/8.0/en/creating-rsa-files-using-openssl.html)

#### 8.3.3.3 使用 openssl 创建 RSA 密钥

本节描述如何使用**openssl**命令设置 RSA 密钥文件，使 MySQL 能够支持通过未加密连接进行安全密码交换的帐户，这些帐户由`sha256_password`和`caching_sha2_password`插件进行身份验证。

注意

有比本文所述过程更简单的生成 RSA 所需文件的替代方法：让服务器自动生成它们或使用**mysql_ssl_rsa_setup**程序（自 MySQL 8.0.34 起已弃用）。请参阅 Section 8.3.3.1, “使用 MySQL 创建 SSL 和 RSA 证书和密钥”。

要创建 RSA 私钥和公钥文件，请在登录用于运行 MySQL 服务器的系统帐户时运行以下命令，以便这些文件归该帐户所有：

```sql
openssl genrsa -out private_key.pem 2048
openssl rsa -in private_key.pem -pubout -out public_key.pem
```

这些命令创建了 2,048 位的密钥。要创建更强大的密钥，请使用更大的值。

然后设置密钥文件的访问模式。私钥应仅由服务器可读，而公钥可以自由分发给客户端用户：

```sql
chmod 400 private_key.pem
chmod 444 public_key.pem
```
