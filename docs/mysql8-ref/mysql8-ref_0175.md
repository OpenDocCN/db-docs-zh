# 6.4.3 mysql_ssl_rsa_setup — 创建 SSL/RSA 文件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-ssl-rsa-setup.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-ssl-rsa-setup.html)

注意

**mysql_ssl_rsa_setup** 在 MySQL 8.0.34 版本中已被弃用。相反，考虑使用 MySQL 服务器在启动时自动生成缺失的 SSL 和 RSA 文件（参见 Automatic SSL and RSA File Generation）。

该程序创建 SSL 证书和密钥文件以及 RSA 密钥对文件，以支持使用 SSL 进行安全连接和使用 RSA 在未加密连接上进行安全密码交换，如果这些文件丢失。**mysql_ssl_rsa_setup** 也可用于创建新的 SSL 文件，如果现有文件已过期。

注意

**mysql_ssl_rsa_setup** 使用 **openssl** 命令，因此��使用取决于您的机器上是否安装了 OpenSSL。

另一种生成 SSL 和 RSA 文件的方法，适用于使用 OpenSSL 编译的 MySQL 发行版，是让服务器自动生成它们。请参阅 Section 8.3.3.1, “Creating SSL and RSA Certificates and Keys using MySQL”。

重要提示

**mysql_ssl_rsa_setup** 通过更容易生成所需文件，降低了使用 SSL 的障碍。然而，由 **mysql_ssl_rsa_setup** 生成的证书是自签名的，安全性不高。在使用 **mysql_ssl_rsa_setup** 生成的文件获得经验后，考虑从注册的证书颁发机构获取 CA 证书。

像这样调用 **mysql_ssl_rsa_setup**：

```sql
mysql_ssl_rsa_setup [*options*]
```

典型的选项包括 `--datadir` 用于指定创建文件的位置，以及 `--verbose` 用于查看 **mysql_ssl_rsa_setup** 执行的 **openssl** 命令。

**mysql_ssl_rsa_setup** 尝试使用默认的文件名创建 SSL 和 RSA 文件。其工作方式如下：

1.  **mysql_ssl_rsa_setup** 检查`PATH`环境变量指定的位置是否存在**openssl**二进制文件。如果未找到**openssl**，**mysql_ssl_rsa_setup** 不执行任何操作。如果找到**openssl**，**mysql_ssl_rsa_setup** 将在 MySQL 数据目录中查找默认 SSL 和 RSA 文件，该目录由`--datadir`选项指定，或者如果未提供`--datadir`选项，则查找编译的数据目录。

1.  **mysql_ssl_rsa_setup** 检查数据目录中具有以下名称的 SSL 文件：

    ```sql
    ca.pem
    server-cert.pem
    server-key.pem
    ```

1.  如果这些文件中有任何一个存在，**mysql_ssl_rsa_setup** 将不创建 SSL 文件。否则，它会调用**openssl**来创建它们，以及一些额外的文件：

    ```sql
    ca.pem               Self-signed CA certificate
    ca-key.pem           CA private key
    server-cert.pem      Server certificate
    server-key.pem       Server private key
    client-cert.pem      Client certificate
    client-key.pem       Client private key
    ```

    这些文件使用 SSL 启用安全的客户端连接；参见第 8.3.1 节，“配置 MySQL 使用加密连接”。

1.  **mysql_ssl_rsa_setup** 检查数据目录中具有以下名称的 RSA 文件：

    ```sql
    private_key.pem      Private member of private/public key pair
    public_key.pem       Public member of private/public key pair
    ```

1.  如果这些文件中有任何一个存在，**mysql_ssl_rsa_setup** 将不创建 RSA 文件。否则，它会调用**openssl**来创建它们。这些文件启用了通过`sha256_password`或`caching_sha2_password`插件进行的帐户的 RSA 在未加密连接上进行安全密码交换；参见第 8.4.1.3 节，“SHA-256 可插拔认证”，以及第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。

有关由**mysql_ssl_rsa_setup**创建的文件特性的信息，请参见第 8.3.3.1 节，“使用 MySQL 创建 SSL 和 RSA 证书和密钥”。

在启动时，如果没有提供除`--ssl`（可能还包括`ssl_cipher`）之外的显式 SSL 选项，则 MySQL 服务器会自动使用由**mysql_ssl_rsa_setup**创建的 SSL 文件来启用 SSL。如果您更喜欢明确指定文件，请在启动时使用`--ssl-ca`、`--ssl-cert`和`--ssl-key`选项来命名`ca.pem`、`server-cert.pem`和`server-key.pem`文件。

服务器还会自动使用由**mysql_ssl_rsa_setup**创建的 RSA 文件来启用 RSA，如果没有提供明确的 RSA 选项。

如果服务器启用了 SSL，则客户端默认使用 SSL 进行连接。要明确指定证书和密钥文件，请使用`--ssl-ca`、`--ssl-cert`和`--ssl-key`选项来命名`ca.pem`、`client-cert.pem`和`client-key.pem`文件。但是，首先可能需要一些额外的客户端设置，因为**mysql_ssl_rsa_setup**默认会在数据目录中创建这些文件。数据目录的权限通常只允许运行 MySQL 服务器的系统帐户访问，因此客户端程序无法使用那里的文件。为了使文件可用，请将它们复制到一个可读（但*不可写*）的目录中供客户端使用：

+   对于本地客户端，可以使用 MySQL 安装目录。例如，如果数据目录是安装目录的子目录，并且您当前位置是数据目录，则可以像这样复制文件：

    ```sql
    cp ca.pem client-cert.pem client-key.pem ..
    ```

+   对于远程客户端，请使用安全通道分发文件，以确保在传输过程中不被篡改。

如果用于 MySQL 安装的 SSL 文件已过期，您可以使用**mysql_ssl_rsa_setup**创建新文件：

1.  停止服务器。

1.  重命名或删除现有的 SSL 文件。您可能希望先备份它们。（RSA 文件不会过期，因此无需删除它们。**mysql_ssl_rsa_setup**会检查它们是否存在，并不会覆盖它们。）

1.  运行**mysql_ssl_rsa_setup**时使用`--datadir`选���来指定新文件的创建位置。

1.  重新启动服务器。

**mysql_ssl_rsa_setup**支持以下命令行选项，可以在命令行或选项文件的`[mysql_ssl_rsa_setup]`和`[mysqld]`组中指定。有关 MySQL 程序使用的选项文件的信息，请参见第 6.2.2.2 节，“使用选项文件”。

**Table 6.10 mysql_ssl_rsa_setup Options**

| 选项名称 | 描述 |
| --- | --- |
| --datadir | 数据目录路径 |
| --help | 显示帮助信息并退出 |
| --suffix | X.509 证书通用名称属性的后�� |
| --uid | 用于文件权限的有效用户名称 |
| --详细 | 详细模式 |
| --version | 显示版本信息并退出 |

+   `--help`, `?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助信息并退出。

+   `--datadir=*`dir_name`*`

    | 命令行格式 | `--datadir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    **mysql_ssl_rsa_setup**应检查默认 SSL 和 RSA 文件的目录路径，并在其中创建文件（如果缺少）。默认为编译的数据目录。

+   `--suffix=*`str`*`

    | 命令行格式 | `--suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    X.509 证书中通用名称属性的后缀。后缀值限制为 17 个字符。默认值基于 MySQL 版本号。

+   `--uid=name`, `-v`

    | 命令行格式 | `--uid=name` |
    | --- | --- |

    应该成为任何创建文件的所有者的用户名称。该值是用户名，而不是数字用户 ID。如果没有此选项，由**mysql_ssl_rsa_setup**创建的文件将由执行它的用户拥有。只有在以`root`身份在支持`chown()`系统调用的系统上执行程序时，此选项才有效。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。显示程序执行的更多输出。例如，程序显示运行的**openssl**命令，并生成输出以指示是否跳过 SSL 或 RSA 文件的创建，因为某些默认文件已经存在。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。
