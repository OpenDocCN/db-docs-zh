> 原文：[`dev.mysql.com/doc/refman/8.0/en/creating-ssl-rsa-files-using-mysql.html`](https://dev.mysql.com/doc/refman/8.0/en/creating-ssl-rsa-files-using-mysql.html)

#### 8.3.3.1 使用 MySQL 创建 SSL 和 RSA 证书和密钥

MySQL 提供了以下方法来创建 SSL 证书和密钥文件以及 RSA 密钥对文件，以支持使用 SSL 进行加密连接和在未加密连接上使用 RSA 进行安全密码交换，如果这些文件丢失：

+   服务器可以在启动时自动生成这些文件，适用于 MySQL 发行版。

+   用户可以手动调用**mysql_ssl_rsa_setup**实用程序（自 MySQL 8.0.34 起已弃用）。

+   对于某些发行类型，如 RPM 和 DEB 软件包，**mysql_ssl_rsa_setup**在数据目录初始化期间调用。在这种情况下，只要**openssl**命令可用，MySQL 发行版不必使用 OpenSSL 进行编译。

重要

服务器自动生成和**mysql_ssl_rsa_setup**有助于降低使用 SSL 的门槛，使生成所需文件变得更容易。然而，通过这些方法生成的证书是自签名的，可能不太安全。在您获得使用这些文件的经验后，考虑从注册的证书颁发机构获取证书/密钥材料。

重要

如果连接到 MySQL 服务器实例的客户端使用带有`extendedKeyUsage`扩展（X.509 v3 扩展）的 SSL 证书，则扩展密钥用途必须包括客户端认证（`clientAuth`）。如果 SSL 证书仅指定用于服务器认证（`serverAuth`）和其他非客户端证书目的，证书验证将失败，客户端连接到 MySQL 服务器实例将失败。MySQL 服务器生成的 SSL 证书中没有`extendedKeyUsage`扩展。如果您使用另一种方式创建的自己的客户端证书，请确保任何`extendedKeyUsage`扩展包括客户端认证。

+   自动 SSL 和 RSA 文件生成

+   使用 mysql_ssl_rsa_setup 手动生成 SSL 和 RSA 文件

+   SSL 和 RSA 文件特性

##### 自动 SSL 和 RSA 文件生成

对于使用 OpenSSL 编译的 MySQL 发行版，MySQL 服务器具有在启动时自动生成缺失的 SSL 和 RSA 文件的能力。`auto_generate_certs`、`sha256_password_auto_generate_rsa_keys`和`caching_sha2_password_auto_generate_rsa_keys`系统变量控制这些文件的自动生成。这些变量默认启用。它们可以在启动时启用和检查，但不能在运行时设置。

在启动时，如果启用了`auto_generate_certs`系统变量，没有指定除`--ssl`之外的其他 SSL 选项，并且数据目录中缺少服务器端 SSL 文件，则服务器会自动在数据目录中生成服务器端和客户端 SSL 证书和密钥文件。这些文件使得可以使用 SSL 加密客户端连接；参见第 8.3.1 节，“配置 MySQL 使用加密连接”。

1.  服务器检查数据目录中具有以下名称的 SSL 文件：

    ```sql
    ca.pem
    server-cert.pem
    server-key.pem
    ```

1.  如果存在任何这些文件，则服务器不会创建 SSL 文件。否则，它会创建它们，以及一些额外的文件：

    ```sql
    ca.pem               Self-signed CA certificate
    ca-key.pem           CA private key
    server-cert.pem      Server certificate
    server-key.pem       Server private key
    client-cert.pem      Client certificate
    client-key.pem       Client private key
    ```

1.  如果服务器自动生成 SSL 文件，则使用`ca.pem`、`server-cert.pem`和`server-key.pem`文件的名称来设置相应的系统变量（`ssl_ca`、`ssl_cert`、`ssl_key`）。

在启动时，如果满足以下所有条件，则服务器会自动在数据目录中生成 RSA 私钥/公钥对文件：启用了`sha256_password_auto_generate_rsa_keys`或`caching_sha2_password_auto_generate_rsa_keys`系统变量；未指定 RSA 选项；数据目录中缺少 RSA 文件。这些密钥对文件使得可以在通过`sha256_password`或`caching_sha2_password`插件进行身份验证的帐户之间使用 RSA 在未加密连接上进行安全密码交换；参见第 8.4.1.3 节，“SHA-256 可插拔认证”和第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

1.  服务器检查数据目录中具有以下名称的 RSA 文件：

    ```sql
    private_key.pem      Private member of private/public key pair
    public_key.pem       Public member of private/public key pair
    ```

1.  如果存在任何这些文件，则服务器不会创建 RSA 文件。否则，它会创建它们。

1.  如果服务器自动生成 RSA 文件，则使用它们的名称来设置相应的系统变量(`sha256_password_private_key_path`和`sha256_password_public_key_path`; `caching_sha2_password_private_key_path`和`caching_sha2_password_public_key_path`)。

##### 使用 mysql_ssl_rsa_setup 手动生成 SSL 和 RSA 文件

MySQL 发行版包含一个**mysql_ssl_rsa_setup**实用程序（自 MySQL 8.0.34 起已弃用），可以手动调用以生成 SSL 和 RSA 文件。该实用程序包含在所有 MySQL 发行版中，但需要**openssl**命令可用。有关使用说明，请参见第 6.4.3 节，“mysql_ssl_rsa_setup — Create SSL/RSA Files”。

##### SSL 和 RSA 文件特征

服务器自动创建的 SSL 和 RSA 文件或通过调用**mysql_ssl_rsa_setup**创建的文件具有以下特征：

+   SSL 和 RSA 的位数为 2048 位。

+   SSL CA 证书是自签名的。

+   SSL 服务器和客户端证书使用 CA 证书和密钥签名，使用`sha256WithRSAEncryption`签名算法。

+   SSL 证书使用这些通用名称（CN）值，具有适当的证书类型（CA、服务器、客户端）：

    ```sql
    ca.pem:         MySQL_Server_*suffix*_Auto_Generated_CA_Certificate
    server-cert.pm: MySQL_Server_*suffix*_Auto_Generated_Server_Certificate
    client-cert.pm: MySQL_Server_*suffix*_Auto_Generated_Client_Certificate
    ```

    *`suffix`*值基于 MySQL 版本号。对于由**mysql_ssl_rsa_setup**生成的文件，可以使用`--suffix`选项显式指定后缀。

    对于服务器生成的文件，如果结果 CN 值超过 64 个字符，则名称的`_*`后缀部分将被省略。

+   SSL 文件的国家（C）、州或省（ST）、组织（O）、组织单位名称（OU）和电子邮件地址为空值。

+   由服务器或**mysql_ssl_rsa_setup**生成的 SSL 文件从生成时起有效期为十年。

+   RSA 文件不会过期。

+   SSL 文件对于每个证书/密钥对具有不同的序列号（CA 为 1，服务器为 2，客户端为 3）。

+   服务器自动创建的文件归属于运行服务器的账户。使用 **mysql_ssl_rsa_setup** 创建的文件归属于调用该程序的用户。在支持 `chown()` 系统调用的系统上，如果程序由 `root` 调用并且提供了 `--uid` 选项来指定应该拥有这些文件的用户，则可以更改这一点。

+   在 Unix 和类 Unix 系统上，证书文件的文件访问模式为 644（即，全局可读），密钥文件的文件访问模式为 600（即，只能由运行服务器的账户访问）。

要查看 SSL 证书的内容（例如，检查其有效日期范围），可以直接调用 **openssl**：

```sql
openssl x509 -text -in ca.pem
openssl x509 -text -in server-cert.pem
openssl x509 -text -in client-cert.pem
```

使用以下 SQL 语句也可以检查 SSL 证书的过期信息：

```sql
mysql> SHOW STATUS LIKE 'Ssl_server_not%';
+-----------------------+--------------------------+
| Variable_name         | Value                    |
+-----------------------+--------------------------+
| Ssl_server_not_after  | Apr 28 14:16:39 2027 GMT |
| Ssl_server_not_before | May  1 14:16:39 2017 GMT |
+-----------------------+--------------------------+
```
