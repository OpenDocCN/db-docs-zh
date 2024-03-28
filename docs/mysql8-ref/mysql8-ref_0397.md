> 原文：[`dev.mysql.com/doc/refman/8.0/en/sha256-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/sha256-pluggable-authentication.html)

#### 8.4.1.3 SHA-256 可插拔认证

MySQL 提供了两个认证插件，用于实现 SHA-256 哈希算法对用户账户密码进行加密：

+   `sha256_password`：实现基本的 SHA-256 认证。

+   `caching_sha2_password`：实现 SHA-256 认证（类似于`sha256_password`），但在服务器端使用缓存以提高性能，并具有更广泛适用性的附加功能。

本节描述了原始的非缓存 SHA-2 认证插件。有关缓存插件的信息，请参见第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

重要提示

在 MySQL 8.0 中，默认认证插件为`caching_sha2_password`而不是`mysql_native_password`。有关此更改对服务器操作以及服务器与客户端和连接器的兼容性的影响的信息，请参见 caching_sha2_password 作为首选认证插件。

由于`caching_sha2_password`是 MySQL 8.0 中的默认认证插件，并提供了`sha256_password`认证插件的全部功能，因此`sha256_password`已被弃用；预计在未来的 MySQL 版本中将其移除。使用`sha256_password`进行认证的 MySQL 账户应迁移到使用`caching_sha2_password`。

重要提示

要使用使用`sha256_password`插件进行认证连接到服务器，您必须使用支持密码交换的 TLS 连接或 RSA 密钥对的非加密连接，如本节后面描述的那样。无论哪种方式，`sha256_password`插件都使用 MySQL 的加密功能。参见第 8.3 节，“使用加密连接”。

注意

在`sha256_password`中，“sha256”指的是插件用于加密的 256 位摘要长度。在`caching_sha2_password`中，“sha2”更普遍地指 SHA-2 类加密算法，256 位加密是其中的一个实例。后者的名称选择为未来扩展可能的摘要长度留出了空间，而不更改插件名称。

以下表格显示了服务器端和客户端端的插件名称。

**表 8.18 SHA-256 认证的插件和库名称**

| 插件或文件 | 插件或文件名 |
| --- | --- |
| 服务器端插件 | `sha256_password` |
| 客户端端插件 | `sha256_password` |
| 库文件 | 无（插件已内置） |

以下各节提供了特定于 SHA-256 可插拔认证的安装和使用信息：

+   安装 SHA-256 可插拔认证

+   使用 SHA-256 可插拔认证

有关 MySQL 中可插拔认证的一般信息，请参见第 8.2.17 节，“可插拔认证”。

##### 安装 SHA-256 可插拔认证

`sha256_password`插件以服务器和客户端形式存在：

+   服务器端插件内置于服务器中，无需显式加载，并且无法通过卸载来禁用。

+   客户端端插件内置于`libmysqlclient`客户端库中，并可供链接到`libmysqlclient`的任何程序使用。

##### 使用 SHA-256 可插拔认证

要设置使用`sha256_password`插件进行 SHA-256 密码哈希的帐户，请使用以下语句，其中*`password`*是所需的帐户密码：

```sql
CREATE USER 'sha256user'@'localhost'
IDENTIFIED WITH sha256_password BY '*password*';
```

服务器将`sha256_password`插件分配给帐户，并使用它使用 SHA-256 加密密码，将这些值存储在`mysql.user`系统表的`plugin`和`authentication_string`列中。

上述说明不假设`sha256_password`是默认认证插件。如果`sha256_password`是默认认证插件，则可以使用更简单的`CREATE USER`语法。

要将服务器的默认认证插件设置为`sha256_password`启动服务器，请在服务器选项文件中添加以下行：

```sql
[mysqld]
default_authentication_plugin=sha256_password
```

这将导致默认情况下新帐户使用`sha256_password`插件。因此，可以创建帐户并设置其密码而无需明确命名插件：

```sql
CREATE USER 'sha256user'@'localhost' IDENTIFIED BY '*password*';
```

将`default_authentication_plugin`设置为`sha256_password`的另一个后果是，要使用其他插件进行帐户创建，必须明确指定该插件。例如，要使用`mysql_native_password`插件，请使用以下语句：

```sql
CREATE USER 'nativeuser'@'localhost'
IDENTIFIED WITH mysql_native_password BY '*password*';
```

`sha256_password`支持通过安全传输进行连接。如果 MySQL 使用 OpenSSL 编译，并且要连接的 MySQL 服务器配置为支持 RSA（使用本节后面给出的 RSA 配置过程），`sha256_password`还支持在未加密连接上使用 RSA 进行加密密码交换。

RSA 支持具有以下特征：

+   在服务器端，两个系统变量命名 RSA 私钥和公钥对文件：`sha256_password_private_key_path`和`sha256_password_public_key_path`。如果要使用与系统变量默认值不同的名称的密钥文件，数据库管理员必须在服务器启动时设置这些变量。

+   服务器使用`sha256_password_auto_generate_rsa_keys`系统变量来确定是否自动生成 RSA 密钥对文件。请参阅第 8.3.3 节，“创建 SSL 和 RSA 证书和密钥”。

+   `Rsa_public_key`状态变量显示`sha256_password`认证插件使用的 RSA 公钥值。

+   拥有 RSA 公钥的客户端可以在连接过程中与服务器执行基于 RSA 密钥对的密码交换，如后文所述。

+   对于使用`sha256_password`和 RSA 密钥对进行密码交换进行身份验证的帐户的连接，服务器根据需要向客户端发送 RSA 公钥。但是，如果客户端主机上有公钥的副本，则客户端可以使用它来节省客户端/服务器协议中的往返：

    +   对于这些命令行客户端，请使用`--server-public-key-path`选项来指定 RSA 公钥文件：**mysql**, **mysqladmin**, **mysqlbinlog**, **mysqlcheck**, **mysqldump**, **mysqlimport**, **mysqlpump**, **mysqlshow**, **mysqlslap**, **mysqltest**, **mysql_upgrade**.

    +   对于使用 C API 的程序，请调用`mysql_options()`来通过传递`MYSQL_SERVER_PUBLIC_KEY`选项和文件名指定 RSA 公钥文件。

    +   对于副本，使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）与`SOURCE_PUBLIC_KEY_PATH` | `MASTER_PUBLIC_KEY_PATH`选项来指定 RSA 公钥文件。对于组复制，`group_replication_recovery_get_public_key`系统变量具有相同的作用。

对于使用`sha256_password`插件的客户端，在连接到服务器时，密码永远不会以明文形式暴露。密码传输的方式取决于是否使用安全连接或 RSA 加密：

+   如果连接是安全的，则不需要使用 RSA 密钥对，也不会使用。这适用于使用 TLS 加密的连接。密码以明文形式发送，但无法被窃听，因为连接是安全的。

    注意

    与`caching_sha2_password`不同，`sha256_password`插件不将共享内存连接视为安全，即使共享内存传输默认安全。

+   如果连接不安全，并且存在 RSA 密钥对，则连接保持未加密。这适用于未使用 TLS 加密的连接。RSA 仅用于客户端和服务器之间的密码交换，以防止密码窃听。当服务器接收到加密密码时，它会对其进行解密。在加密中使用了一种混淆以防止重复攻击。

+   如果未使用安全连接且不可用 RSA 加密，则连接尝试将失败，因为密码无法在未以明文形式暴露的情况下发送。

注意

要使用`sha256_password`进行 RSA 密码加密，客户端和服务器都必须使用 OpenSSL 进行编译，而不仅仅是其中一个。

假设 MySQL 已使用 OpenSSL 编译，请使用以下过程在客户端连接过程中启用 RSA 密钥对用于密码交换：

1.  使用 Section 8.3.3, “Creating SSL and RSA Certificates and Keys”中的说明创建 RSA 私钥和公钥对文件。

1.  如果私钥和公钥文件位于数据目录中，并且命名为`private_key.pem`和`public_key.pem`（`sha256_password_private_key_path`和`sha256_password_public_key_path`系统变量的默认值），服务器将在启动时自动使用它们。

    否则，要显式命名关键文件，请在服务器选项文件中设置系统变量为关键文件名。如果文件位于服务器数据目录中，则无需指定其完整路径名：

    ```sql
    [mysqld]
    sha256_password_private_key_path=myprivkey.pem
    sha256_password_public_key_path=mypubkey.pem
    ```

    如果密钥文件不位于数据目录中，或者要在系统变量值中明确它们的位置，请使用完整路径名：

    ```sql
    [mysqld]
    sha256_password_private_key_path=/usr/local/mysql/myprivkey.pem
    sha256_password_public_key_path=/usr/local/mysql/mypubkey.pem
    ```

1.  重新启动服务器，然后连接到服务器并检查`Rsa_public_key`状态变量值。实际显示的值与此处显示的值不同，但应为非空：

    ```sql
    mysql> SHOW STATUS LIKE 'Rsa_public_key'\G
    *************************** 1\. row ***************************
    Variable_name: Rsa_public_key
            Value: -----BEGIN PUBLIC KEY-----
    MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDO9nRUDd+KvSZgY7cNBZMNpwX6
    MvE1PbJFXO7u18nJ9lwc99Du/E7lw6CVXw7VKrXPeHbVQUzGyUNkf45Nz/ckaaJa
    aLgJOBCIDmNVnyU54OT/1lcs2xiyfaDMe8fCJ64ZwTnKbY2gkt1IMjUAB5Ogd5kJ
    g8aV7EtKwyhHb0c30QIDAQAB
    -----END PUBLIC KEY-----
    ```

    如果值为空，则服务器发现密钥文件存在问题。请检查错误日志以获取诊断信息。

在服务器配置了 RSA 密钥文件之后，使用`sha256_password`插件进行身份验证的帐户可以选择使用这些密钥文件连接到服务器。如前所述，这些帐户可以使用安全连接（在这种情况下不使用 RSA）或使用 RSA 执行密码交换的未加密连接。假设使用未加密连接。例如：

```sql
$> mysql --ssl-mode=DISABLED -u sha256user -p
Enter password: *password*
```

对于`sha256user`的此连接尝试，服务器确定`sha256_password`是适当的身份验证插件并调用它（因为在`CREATE USER`时指定了该插件）。插件发现连接未加密，因此需要使用 RSA 加密传输密码。在这种情况下，插件将 RSA 公钥发送给客户端，客户端使用它加密密码并将结果返回给服务器。插件在服务器端使用 RSA 私钥解密密码，并根据密码是否正确接受或拒绝连接。

服务器根据需要向客户端发送 RSA 公钥。但是，如果客户端有包含服务器所需的 RSA 公钥的本地副本的文件，可以使用`--server-public-key-path`选项指定该文件：

```sql
$> mysql --ssl-mode=DISABLED -u sha256user -p --server-public-key-path=*file_name*
Enter password: *password*
```

由`--server-public-key-path`选项指定的文件中的公钥值应与由`sha256_password_public_key_path`系统变量指定的服务器端文件中的密钥值相同。如果密钥文件包含有效的公钥值但值不正确，则会发生访问被拒绝的错误。如果密钥文件不包含有效的公钥，则客户端程序无法使用它。在这种情况下，`sha256_password`插件将向客户端发送公钥，就好像没有指定`--server-public-key-path`选项一样。

客户端用户可以通过两种方式获取 RSA 公钥：

+   数据库管理员可以提供公钥文件的副本。

+   客户端用户可以通过其他方式连接到服务器，使用`SHOW STATUS LIKE 'Rsa_public_key'`语句并将返回的密钥值保存在文件中。
