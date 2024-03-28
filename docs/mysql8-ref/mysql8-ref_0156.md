> 原文：[`dev.mysql.com/doc/refman/8.0/en/option-file-options.html`](https://dev.mysql.com/doc/refman/8.0/en/option-file-options.html)

#### 6.2.2.3 影响选项文件处理的命令行选项

大多数支持选项文件的 MySQL 程序处理以下选项。由于这些选项会影响选项文件处理，因此必须在命令行中给出，而不是在选项文件中。为了正常工作，这些选项中的每一个必须在其他选项之前给出，但有以下例外：

+   `--print-defaults` 可以立即在`--defaults-file`、`--defaults-extra-file`或`--login-path`之后使用。

+   在 Windows 上，如果服务器使用`--defaults-file`和`--install`选项启动，则`--install`必须首先。请参见第 2.3.4.8 节，“将 MySQL 作为 Windows 服务启动”。

在指定文件名作为选项值时，避免使用`~` shell 元字符，因为它可能不会按您的预期解释。

**表 6.3 选项文件选项摘要**

| 选项名称 | 描述 |
| --- | --- |
| --defaults-extra-file | 除了通常的选项文件外，还读取指定的选项文件 |
| --defaults-file | 仅读取指定的选项文件 |
| --defaults-group-suffix | 选项组后缀值 |
| --login-path | 从.mylogin.cnf 中读取登录路径选项 |
| --no-defaults | 不读取任何选项文件 |

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=filename` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `[none]` |

    读取此选项文件在全局选项文件之后，但（在 Unix 上）在用户选项文件之前，并且（在所有平台上）在登录路径文件之前。（有关选项文件使用顺序的信息，请参见第 6.2.2.2 节，“使用选项文件”。）如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    请参阅本节开头有关此选项可能指定位置的限制。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=filename` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `[none]` |

    仅读取给定的选项文件。如果文件不存在或无法访问，则会出现错误。*`file_name`*如果作为相对路径名而不是完整路径名给出，则会相对于当前目录进行解释。

    例外情况：即使使用`--defaults-file`，**mysqld**会读取`mysqld-auto.cnf`，客户端程序会读取`.mylogin.cnf`。

    请参阅本节介绍有关此选项可能指定的位置的约束。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    不仅读取通常的选项组，还读取具有通常名称和后缀*`str`*的组。例如，**mysql**客户端通常会读取`[client]`和`[mysql]`组。如果将此选项给出为`--defaults-group-suffix=_other`，**mysql**还会读取`[client_other]`和`[mysql_other]`组。

+   `--login-path=*`name`*`

    | 命令行格式 | `--login-path=name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    从`.mylogin.cnf`登录路径文件中的命名登录路径读取选项。 “登录路径”是一个包含指定要连接到哪个 MySQL 服务器以及要进行身份验证的帐户的选项组。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参阅第 6.6.7 节，“mysql_config_editor — MySQL Configuration Utility”。

    客户端程序读取与命名登录路径对应的选项组，以及程序默认读取的选项组。考虑以下命令：

    ```sql
    mysql --login-path=mypath
    ```

    默认情况下，**mysql**客户端读取`[client]`和`[mysql]`选项组。因此，对于所示的命令，**mysql**从其他选项文件读取`[client]`和`[mysql]`，并从登录路径文件中读取`[client]`、`[mysql]`和`[mypath]`。

    即使使用`--no-defaults`选项，客户端程序也会读取登录路径文件。

    要指定替代的登录路径文件名，请设置`MYSQL_TEST_LOGIN_FILE`环境变量。

    请参阅本节介绍有关此选项可能指定的位置的约束。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，则可以使用`--no-defaults`来防止读取它们。

    例外情况是，即使使用了`--no-defaults`，客户端程序仍会读取`.mylogin.cnf`登录路径文件（如果存在）。这样即使存在`--no-defaults`，也可以以比在命令行上更安全的方式指定密码。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。请参阅第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    打印程序名称以及从选项文件获取的所有选项。密码值被掩码。

    请参阅本节开头有关此选项可能被指定的位置的限制。
