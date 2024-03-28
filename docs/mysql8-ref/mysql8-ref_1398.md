# 19.3.1 设置复制使用加密连接

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-encrypted-connections.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-encrypted-connections.html)

要在复制过程中使用加密连接传输二进制日志，源服务器和副本服务器都必须支持加密网络连接。如果任一服务器不支持加密连接（因为它未被编译或配置为支持），则无法通过加密连接进行复制。

为了为复制设置加密连接，与为客户端/服务器连接设置类似。您必须获取（或创建）一个适当的安全证书，可以在源端使用，并在每个副本上使用来自同一证书颁发机构的类似证书。您还必须获取适当的密钥文件。

有关设置服务器和客户端使用加密连接的更多信息，请参见第 8.3.1 节，“配置 MySQL 使用加密连接”。

要在源端启用加密连接，您必须创建或获取适当的证书和密钥文件，然后将以下配置参数添加到源端 `my.cnf` 文件的 `[mysqld]` 部分中，根据需要更改文件名：

```sql
[mysqld]
ssl_ca=cacert.pem
ssl_cert=server-cert.pem
ssl_key=server-key.pem
```

文件的路径可以是相对的或绝对的；我们建议您始终为此目的使用完整路径。

配置参数如下：

+   `ssl_ca`：证书颁发机构（CA）证书文件的路径名。(`ssl_capath`类似，但指定了 CA 证书文件的目录路径名。)

+   `ssl_cert`：服务器公钥证书文件的路径名。此证书可以发送给客户端，并根据其拥有的 CA 证书进行验证。

+   `ssl_key`：服务器私钥文件的路径名。

要在副本端启用加密连接，使用 `CHANGE REPLICATION SOURCE TO` 语句（MySQL 8.0.23 及更高版本）或 `CHANGE MASTER TO` 语句（MySQL 8.0.23 之前）。

+   要使用 `CHANGE REPLICATION SOURCE TO`（`CHANGE MASTER TO`）命名副本的证书和 SSL 私钥文件，添加适当的 `SOURCE_SSL_*`xxx`*`（`MASTER_SSL_*`xxx`*`）选项，如下所示：

    ```sql
     -> SOURCE_SSL_CA = 'ca_file_name',
     -> SOURCE_SSL_CAPATH = 'ca_directory_name',
     -> SOURCE_SSL_CERT = 'cert_file_name',
     -> SOURCE_SSL_KEY = 'key_file_name',
    ```

    这些选项对应于相同名称的`--ssl-*`xxx`*`选项，如加密连接的命令选项中所述。要使这些选项生效，还必须设置`SOURCE_SSL=1`。对于复制连接，为`SOURCE_SSL_CA`或`SOURCE_SSL_CAPATH`中的任一值指定一个值相当于设置`--ssl-mode=VERIFY_CA`。只有在使用指定信息找到有效匹配的证书颁发机构（CA）证书时，连接尝试才会成功。

+   要激活主机名身份验证，添加`SOURCE_SSL_VERIFY_SERVER_CERT`选项，如下所示：

    ```sql
     -> SOURCE_SSL_VERIFY_SERVER_CERT=1,
    ```

    此选项对应于`--ssl-verify-server-cert`选项，在 MySQL 5.7 中已弃用，并在 MySQL 8.0 中删除。对于复制连接，指定`MASTER_SSL_VERIFY_SERVER_CERT=1`相当于设置`--ssl-mode=VERIFY_IDENTITY`，如加密连接的命令选项中所述。要使此选项生效，还必须设置`SOURCE_SSL=1`。主机名身份验证无法与自签名证书一起使用。

+   要激活证书吊销列表（CRL）检查，添加`SOURCE_SSL_CRL`或`SOURCE_SSL_CRLPATH`选项，如下所示：

    ```sql
     -> SOURCE_SSL_CRL = 'crl_file_name',
     -> SOURCE_SSL_CRLPATH = 'crl_directory_name',
    ```

    这些选项对应于相同名称的`--ssl-*`xxx`*`选项，如加密连接的命令选项中所述。如果未指定，将不进行 CRL 检查。

+   要指定副本允许的密码、密码套件和加密协议列表，使用`SOURCE_SSL_CIPHER`、`SOURCE_TLS_VERSION`和`SOURCE_TLS_CIPHERSUITES`选项，如下所示：

    ```sql
     -> SOURCE_SSL_CIPHER = 'cipher_list',
     -> SOURCE_TLS_VERSION = 'protocol_list',
     -> SOURCE_TLS_CIPHERSUITES = 'ciphersuite_list',
    ```

    +   `SOURCE_SSL_CIPHER`选项指定了一个由冒号分隔的一个或多个副本允许的密码。

    +   `SOURCE_TLS_VERSION`选项指定了一个由逗号分隔的 TLS 加密协议列表，副本允许用于复制连接，格式类似于`tls_version`服务器系统变量。连接过程协商使用源和副本都允许的最高 TLS 版本。为了能够连接，副本必须至少与源有一个 TLS 版本相同。

    +   `SOURCE_TLS_CIPHERSUITES`选项（从 MySQL 8.0.19 开始提供）指定了一个以冒号分隔的允许副本使用的一个或多个密码套件列表，如果连接使用 TLSv1.3。如果在使用 TLSv1.3 时将此选项设置为`NULL`（如果您没有设置该选项，则默认为此），则允许启用默认启用的密码套件。如果将选项设置为空字符串，则不允许任何密码套件，并且因此不使用 TLSv1.3。

    您可以在这些列表中指定的协议、密码和密码套件取决于用于编译 MySQL 的 SSL 库。有关格式、允许的值以及如果不指定选项时的默认值的信息，请参阅第 8.3.2 节，“加密连接 TLS 协议和密码”。

    注意

    在 MySQL 8.0.16 到 8.0.18 中，MySQL 支持 TLSv1.3，但`SOURCE_TLS_CIPHERSUITES`选项不可用。在这些版本中，如果源和副本之间的连接使用 TLSv1.3，则源必须允许至少一个默认启用的 TLSv1.3 密码套件的使用。从 MySQL 8.0.19 开始，您可以使用该选项指定任何密码套件的选择，包括仅非默认密码套件（如果需要）。

+   更新源信息后，在副本上启动复制过程，如下所示：

    ```sql
    mysql> START SLAVE;
    ```

    从 MySQL 8.0.22 开始，首选使用`START REPLICA`，如下所示：

    ```sql
    mysql> START REPLICA;
    ```

    你可以使用`SHOW REPLICA STATUS`（在 MySQL 8.0.22 之前，`SHOW SLAVE STATUS`）语句来确认已成功建立加密连接。

+   在副本上要求加密连接并不意味着源要求来自副本的加密连接。如果要确保源仅接受使用加密连接连接的副本，可以在源上创建一个使用`REQUIRE SSL`选项的复制用户帐户，然后授予该用户`REPLICATION SLAVE`权限。例如：

    ```sql
    mysql> CREATE USER 'repl'@'%.example.com' IDENTIFIED BY '*password*'
     -> REQUIRE SSL;
    mysql> GRANT REPLICATION SLAVE ON *.*
     -> TO 'repl'@'%.example.com';
    ```

    如果您在源上有现有的复制用户帐户，可以使用以下语句向其添加`REQUIRE SSL`：

    ```sql
    mysql> ALTER USER 'repl'@'%.example.com' REQUIRE SSL;
    ```
