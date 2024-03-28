> 原文：[`dev.mysql.com/doc/refman/8.0/en/caching-sha2-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/caching-sha2-pluggable-authentication.html)

#### 8.4.1.2 缓存 SHA-2 可插拔身份验证

MySQL 提供了两个实现 SHA-256 哈希用于用户帐户密码的身份验证插件：

+   `sha256_password`：实现基本的 SHA-256 身份验证。

+   `caching_sha2_password`：实现 SHA-256 身份验证（类似于`sha256_password`），但在服务器端使用缓存以提高性能，并具有更广泛适用性的附加功能。

本节描述了缓存 SHA-2 身份验证插件。有关原始基本（非缓存）插件的信息，请参见第 8.4.1.3 节，“SHA-256 可插拔身份验证”。

重要

在 MySQL 8.0 中，`caching_sha2_password`是默认的身份验证插件，而不是`mysql_native_password`。有关此更改对服务器操作以及服务器与客户端和连接器的兼容性的影响的信息，请参见将 caching_sha2_password 作为首选身份验证插件。

重要

要使用使用`caching_sha2_password`插件进行身份验证的帐户连接到服务器，必须使用安全连接或支持使用 RSA 密钥对进行密码交换的未加密连接，如本节后面所述。无论哪种方式，`caching_sha2_password`插件都使用 MySQL 的加密功能。参见第 8.3 节，“使用加密连接”。

注意

在`sha256_password`中，“sha256”指的是插件用于加密的 256 位摘要长度。在`caching_sha2_password`中，“sha2”更普遍地指 SHA-2 类加密算法，256 位加密是其中的一个实例。后者的名称选择为未来扩展可能的摘要长度留出了空间，而无需更改插件名称。

与`sha256_password`相比，`caching_sha2_password`插件具有以下优点：

+   在服务器端，内存中的缓存使得已连接过的用户再次连接时能够更快地重新验证身份。

+   基于 RSA 的密码交换可用，无论 MySQL 链接到哪个 SSL 库。

+   支持使用 Unix 套接字文件和共享内存协议的客户端连接。

以下表格显示了服务器端和客户端的插件名称。

**表 8.17 SHA-2 身份验证的插件和库名称**

| 插件或文件 | 插件或文件名 |
| --- | --- |
| 服务器端插件 | `caching_sha2_password` |
| 客户端插件 | `caching_sha2_password` |
| 库文件 | 无（插件内置） |

以下各节提供了特定于缓存 SHA-2 可插拔身份验证的安装和使用信息：

+   安装 SHA-2 可插拔身份验证

+   使用 SHA-2 可插拔身份验证

+   SHA-2 可插拔身份验证的缓存操作

有关 MySQL 中可插拔身份验证的一般信息，请参阅第 8.2.17 节，“可插拔身份验证”。

##### 安装 SHA-2 可插拔身份验证

`caching_sha2_password`插件以服务器和客户端形式存在：

+   服务器端插件内置于服务器中，无需显式加载，并且无法通过卸载来禁用。

+   客户端插件内置于`libmysqlclient`客户端库中，并可供链接到`libmysqlclient`的任何程序使用。

服务器端插件使用`sha2_cache_cleaner`审计插件作为辅助工具来执行密码缓存管理。`sha2_cache_cleaner`与`caching_sha2_password`一样，是内置的，无需安装。

##### 使用 SHA-2 可插拔身份验证

要设置一个使用`caching_sha2_password`插件进行 SHA-256 密码哈希的帐户，请使用以下语句，其中*`password`*是所需的帐户密码：

```sql
CREATE USER 'sha2user'@'localhost'
IDENTIFIED WITH caching_sha2_password BY '*password*';
```

服务器将`caching_sha2_password`插件分配给帐户，并使用它使用 SHA-256 加密密码，将这些值存储在`mysql.user`系统表的`plugin`和`authentication_string`列中。

前述说明并不假设`caching_sha2_password`是默认的身份验证插件。如果`caching_sha2_password`是默认的身份验证插件，则可以使用更简单的`CREATE USER`语法。

要将服务器的默认身份验证插件设置为`caching_sha2_password`，请在服务器选项文件中添加以下行：

```sql
[mysqld]
default_authentication_plugin=caching_sha2_password
```

这将导致`caching_sha2_password`插件默认用于新帐户。因此，可以创建帐户并设置其密码而无需显式命名插件：

```sql
CREATE USER 'sha2user'@'localhost' IDENTIFIED BY '*password*';
```

将`default_authentication_plugin`设置为`caching_sha2_password`的另一个后果是，要使用其他插件进行帐户创建，必须明确指定该插件。例如，要使用`mysql_native_password`插件，请使用以下语句：

```sql
CREATE USER 'nativeuser'@'localhost'
IDENTIFIED WITH mysql_native_password BY '*password*';
```

`caching_sha2_password` 支持通过安全传输进行连接。如果您按照本节后面给出的 RSA 配置过程，它还支持在未加密连接上使用 RSA 进行加密密码交换。RSA 支持具有以下特点：

+   在服务器端，两个系统变量命名了 RSA 私钥和公钥对文件：`caching_sha2_password_private_key_path` 和 `caching_sha2_password_public_key_path`。如果要使用与系统变量默认值不同的名称的密钥文件，则数据库管理员必须在服务器启动时设置这些变量。

+   服务器使用 `caching_sha2_password_auto_generate_rsa_keys` 系统变量来确定是否自动生成 RSA 密钥对文件。请参阅 Section 8.3.3, “Creating SSL and RSA Certificates and Keys”。

+   `Caching_sha2_password_rsa_public_key` 状态变量显示了 `caching_sha2_password` 认证插件使用的 RSA 公钥值。

+   拥有 RSA 公钥的客户端可以在连接过程中与服务器执行基于 RSA 密钥对的密码交换，如后面所述。

+   对于使用 `caching_sha2_password` 和 RSA 密钥对进行密码交换进行身份验证的帐户的连接，默认情况下服务器不会向客户端发送 RSA 公钥。客户端可以使用所需公钥的客户端副本，或者从服务器请求公钥。

    使用受信任的本地公钥副本使客户端能够避免在客户端/服务器协议中的往返，并且比从服务器请求公钥更安全。另一方面，从服务器请求公钥更方便（不需要管理客户端文件）并且在安全网络环境中可能是可接受的。

    +   对于命令行客户端，请使用`--server-public-key-path`选项来指定 RSA 公钥文件。使用`--get-server-public-key`选项从服务器请求公钥。以下程序支持这两个选项：**mysql**、**mysqlsh**、**mysqladmin**、**mysqlbinlog**、**mysqlcheck**、**mysqldump**、**mysqlimport**、**mysqlpump**、**mysqlshow**、**mysqlslap**、**mysqltest**、**mysql_upgrade**。

    +   对于使用 C API 的程序，通过调用`mysql_options()`来指定 RSA 公钥文件，传递`MYSQL_SERVER_PUBLIC_KEY`选项和文件名，或者通过传递`MYSQL_OPT_GET_SERVER_PUBLIC_KEY`选项从服务器请求公钥。

    +   对于副本，使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）与`SOURCE_PUBLIC_KEY_PATH` | `MASTER_PUBLIC_KEY_PATH`选项来指定 RSA 公钥文件，或使用`GET_SOURCE_PUBLIC_KEY` | `GET_MASTER_PUBLIC_KEY`选项从源请求公钥。对于组复制，`group_replication_recovery_public_key_path`和`group_replication_recovery_get_public_key`系统变量具有相同的作用。

    在所有情况下，如果给定指定有效公钥文件的选项，则该选项优先于从服务器请求公钥的选项。

对于使用`caching_sha2_password`插件的客户端，在连接到服务器时，密码永远不会以明文形式暴露。密码传输的方式取决于是否使用安全连接或 RSA 加密：

+   如果连接是安全的，则不需要使用 RSA 密钥对。这适用于使用 TLS 加密的 TCP 连接，以及 Unix 套接字文件和共享内存连接。密码以明文形式发送，但由于连接是安全的，无法窃听。

+   如果连接不安全，则使用 RSA 密钥对。这适用于未使用 TLS 加密的 TCP 连接和命名管道连接。RSA 仅用于客户端和服务器之间的密码交换，以防止密码窗口。当服务器接收到加密密码时，它会对其进行解密。加密中使用了混淆以防止重复攻击。

要在客户端连接过程中使用 RSA 密钥对进行密码交换，请使用以下步骤：

1.  使用 Section 8.3.3, “Creating SSL and RSA Certificates and Keys”中的说明创建 RSA 私钥和公钥对文件。

1.  如果私钥和公钥文件位于数据目录中，并且命名为`private_key.pem`和`public_key.pem`（`caching_sha2_password_private_key_path`和`caching_sha2_password_public_key_path`系统变量的默认值），服务器将在启动时自动使用它们。

    否则，要显式命名密钥文件，请在服务器选项文件中将系统变量设置为密钥文件名。如果文件位于服务器数据目录中，则无需指定其完整路径名：

    ```sql
    [mysqld]
    caching_sha2_password_private_key_path=myprivkey.pem
    caching_sha2_password_public_key_path=mypubkey.pem
    ```

    如果密钥文件不位于数据目录中，或者要在系统变量值中明确指定其位置，请使用完整路径名：

    ```sql
    [mysqld]
    caching_sha2_password_private_key_path=/usr/local/mysql/myprivkey.pem
    caching_sha2_password_public_key_path=/usr/local/mysql/mypubkey.pem
    ```

1.  如果要更改`caching_sha2_password`在密码生成过程中使用的哈希轮数，请设置`caching_sha2_password_digest_rounds`系统变量。例如：

    ```sql
    [mysqld]
    caching_sha2_password_digest_rounds=10000
    ```

1.  重新启动服务器，然后连接到服务器并检查`Caching_sha2_password_rsa_public_key`状态变量值。实际显示的值与此处显示的值不同，但应为非空：

    ```sql
    mysql> SHOW STATUS LIKE 'Caching_sha2_password_rsa_public_key'\G
    *************************** 1\. row ***************************
    Variable_name: Caching_sha2_password_rsa_public_key
            Value: -----BEGIN PUBLIC KEY-----
    MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDO9nRUDd+KvSZgY7cNBZMNpwX6
    MvE1PbJFXO7u18nJ9lwc99Du/E7lw6CVXw7VKrXPeHbVQUzGyUNkf45Nz/ckaaJa
    aLgJOBCIDmNVnyU54OT/1lcs2xiyfaDMe8fCJ64ZwTnKbY2gkt1IMjUAB5Ogd5kJ
    g8aV7EtKwyhHb0c30QIDAQAB
    -----END PUBLIC KEY-----
    ```

    如果值为空，则服务器发现密钥文件存在问题。检查错误日志以获取诊断信息。

在服务器配置了 RSA 密钥文件后，使用`caching_sha2_password`插件进行身份验证的帐户可以选择使用这些密钥文件连接到服务器。如前所述，这些帐户可以使用安全连接（在这种情况下不使用 RSA）或使用 RSA 执行密码交换的未加密连接。假设使用未加密连接。例如：

```sql
$> mysql --ssl-mode=DISABLED -u sha2user -p
Enter password: *password*
```

对于 `sha2user` 的此连接尝试，服务器确定 `caching_sha2_password` 是适当的认证插件并调用它（因为在 `CREATE USER` 时指定了该插件）。插件发现连接未加密，因此需要使用 RSA 加密传输密码。然而，服务器不会将公钥发送给客户端，客户端也未提供公钥，因此无法加密密码，连接失败：

```sql
ERROR 2061 (HY000): Authentication plugin 'caching_sha2_password'
reported error: Authentication requires secure connection.
```

要从服务器请求 RSA 公钥，请指定 `--get-server-public-key` 选项：

```sql
$> mysql --ssl-mode=DISABLED -u sha2user -p --get-server-public-key
Enter password: *password*
```

在这种情况下，服务器将 RSA 公钥发送给客户端，客户端使用它加密密码并将结果返回给服务器。插件在服务器端使用 RSA 私钥解密密码，并根据密码是否正确接受或拒绝连接。

或者，如果客户端有包含服务器所需的 RSA 公钥的本地副本的文件，则可以使用 `--server-public-key-path` 选项指定该文件：

```sql
$> mysql --ssl-mode=DISABLED -u sha2user -p --server-public-key-path=*file_name*
Enter password: *password*
```

在这种情况下，客户端使用公钥加密密码并将结果返回给服务器。插件在服务器端使用 RSA 私钥解密密码，并根据密码是否正确接受或拒绝连接。

`--server-public-key-path` 选项指定的文件中的公钥值应与由 `caching_sha2_password_public_key_path` 系统变量指定的服务器端文件中的密钥值相同。如果密钥文件包含有效的公钥值但该值不正确，则会出现访问被拒绝的错误。如果密钥文件不包含有效的公钥，则客户端程序无法使用它。

客户端用户可以通过两种方式获取 RSA 公钥：

+   数据库管理员可以提供公钥文件的副本。

+   可以连接到服务器的客户端用户可以使用 `SHOW STATUS LIKE 'Caching_sha2_password_rsa_public_key'` 语句并将返回的密钥值保存在文件中。

##### SHA-2 可插拔认证的缓存操作

在服务器端，`caching_sha2_password` 插件使用内存中的缓存来更快地对先前连接过的客户端进行认证。条目由账户名/密码哈希对组成。缓存的工作方式如下：

1.  当客户端连接时，`caching_sha2_password` 检查客户端和密码是否与某个缓存条目匹配。如果匹配，则认证成功。

1.  如果没有匹配的缓存条目，插件会尝试根据`mysql.user`系统表中的凭证验证客户端。如果成功，`caching_sha2_password`会为客户端添加一个条目到哈希中。否则，认证失败并拒绝连接。

这样，当客户端首次连接时，会对`mysql.user`系统表进行认证。当客户端随后连接时，会对缓存进行更快的认证。

除了添加条目之外的密码缓存操作由`sha2_cache_cleaner`审计插件处理，该插件代表`caching_sha2_password`执行这些操作：

+   它清除任何被重命名或删除的账户的缓存条目，或者更改了凭证或认证插件的账户。

+   当执行`FLUSH PRIVILEGES`语句时，会清空缓存。

+   在服务器关闭时会清空缓存。（这意味着缓存在服务器重新启动时不是持久的。）

清空缓存操作会影响后续客户端连接的认证要求。对于每个用户账户，在以下任何操作之后，用户的第一个客户端连接必须使用安全连接（使用 TLS 凭证的 TCP 连接，Unix 套接字文件或共享内存）或基于 RSA 密钥对的密码交换：

+   在账户创建后。

+   在账户密码更改后。

+   在`RENAME USER`后的账户。

+   在`FLUSH PRIVILEGES`后。

`FLUSH PRIVILEGES`清空整个缓存并影响所有使用`caching_sha2_password`插件的账户。其他操作清空特定缓存条目并仅影响操作中的账户。

一旦用户成功认证，账户将被输入缓存，后续连接不需要安全连接或 RSA 密钥对，直到发生影响该账户的另一个清空缓存事件。（当可以使用缓存时，服务器使用一种不使用明文密码传输且不需要安全连接的挑战-响应机制。）
