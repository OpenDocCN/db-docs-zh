> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-options.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-options.html)

#### 8.4.4.18 密钥环命令选项

MySQL 支持以下与密钥环相关的命令行选项：

+   `--keyring-migration-destination=*`plugin`*`

    | 命令行格式 | `--keyring-migration-destination=plugin_name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于密钥迁移的目标密钥环插件。参见第 8.4.4.14 节，“在密钥环密钥存储之间迁移密钥”。选项值的解释取决于是否指定了`--keyring-migration-to-component`：

    +   如果没有指定，则选项值是一个密钥环插件，解释方式与`--keyring-migration-source`相同。

    +   如果是，则选项值是一个密钥环组件，指定为插件目录中的组件库名称，包括任何特定于平台的扩展，如`.so`或`.dll`。

    注意

    `--keyring-migration-source`和`--keyring-migration-destination`对于所有密钥环迁移操作都是必需的。源和目标插件必须不同，并且迁移服务器必须支持这两个插件。

+   `--keyring-migration-host=*`host_name`*`

    | 命令行格式 | `--keyring-migration-host=host_name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `localhost` |

    当前正在使用其中一个密钥迁移密钥库的运行服务器的主机位置。参见第 8.4.4.14 节，“在密钥环密钥存储之间迁移密钥”。迁移始终在本地主机上进行，因此该选项始终指定用于连接到本地服务器的值，例如`localhost`、`127.0.0.1`、`::1`或本地主机 IP 地址或主机名。

+   [`--keyring-migration-password[=*`password`*]`](keyring-options.html#option_mysqld_keyring-migration-password)

    | 命令行格式 | `--keyring-migration-password[=password]` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接到当前使用其中一个密钥迁移密钥库的运行服务器的 MySQL 帐户的密码。参见第 8.4.4.14 节，“在密钥环密钥存储之间迁移密钥”。

    密码值是可选的。如果未提供，则服务器会提示输入。如果提供了密码，则`--keyring-migration-password=`后面必须*没有空格*。如果未指定密码选项，则默认情况下不发送密码。

    在命令行上指定密码应被视为不安全。请参见第 8.1.2.1 节，“密码安全的最终用户指南”。您可以使用选项文件来避免在命令行上提供密码。在这种情况下，文件应具有严格的模式，并且只能由用于运行迁移服务器的帐户访问。

+   `--keyring-migration-port=*`端口号`*`

    | 命令行格式 | `--keyring-migration-port=端口号` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `3306` |

    对于 TCP/IP 连接，连接到当前使用密钥迁移密钥库之一的运行服务器的端口号。参见第 8.4.4.14 节，“在密钥环密钥存储之间迁移密钥”。

+   `--keyring-migration-socket=*`路径`*`

    | 命令行格式 | `--keyring-migration-socket={file_name&#124;pipe_name}` |
    | --- | --- |
    | 类型 | 字符串 |

    对于 Unix 套接字文件或 Windows 命名管道连接，连接到当前使用密钥迁移密钥库之一的运行服务器的套接字文件或命名管道。参见第 8.4.4.14 节，“在密钥环密钥存储之间迁移密钥”。

+   `--keyring-migration-source=*`插件`*`

    | 命令行格式 | `--keyring-migration-source=plugin_name` |
    | --- | --- |
    | 类型 | 字符串 |

    密钥迁移的源密钥环插件。参见第 8.4.4.14 节，“在密钥环密钥存储之间迁移密钥”。

    该选项的值类似于`--plugin-load`，但只能指定一个插件库。该值以*`plugin_library`*或*`name`*`=`*`plugin_library`*的形式给出，其中*`plugin_library`*是包含插件代码的库文件的名称，*`name`*是要加载的插件的名称。如果命名了一个插件库而没有任何前置插件名称，服务器将加载库中的所有插件。有了前置插件名称，服务器将仅从库中加载指定的插件。服务器在由`plugin_dir`系统变量命名的目录中查找插件库文件。

    注意

    `--keyring-migration-source`和`--keyring-migration-destination`对于所有密钥环迁移操作都是必需的。源和目标插件必须不同，并且迁移服务器必须支持这两个插件。

+   `--keyring-migration-to-component`

    | 命令行格式 | `--keyring-migration-to-component[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.24 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    表示密钥迁移是从密钥环插件到密钥环组件的。此选项使得可以从任何密钥环插件迁移密钥到任何密钥环组件，从而便于将 MySQL 安装从密钥环插件过渡到密钥环组件。

    对于从一个密钥环组件迁移密钥到另一个密钥环组件，请使用**mysql_migrate_keyring**实用程序。不支持从密钥环组件迁移到密钥环插件。请参阅第 8.4.4.14 节，“在密钥环密钥库之间迁移密钥”。

+   `--keyring-migration-user=*`user_name`*`

    | 命令行格式 | `--keyring-migration-user=user_name` |
    | --- | --- |
    | 类型 | 字符串 |

    用于连接当前正在使用其中一个密钥迁移密钥库的运行服务器的 MySQL 帐户的用户名。请参阅第 8.4.4.14 节，“在密钥环密钥库之间迁移密钥”。
