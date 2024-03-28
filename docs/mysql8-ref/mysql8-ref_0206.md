# 6.6.7 mysql_config_editor — MySQL Configuration Utility

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-config-editor.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-config-editor.html)

**mysql_config_editor** 实用程序允许您将身份验证凭据存储在名为`.mylogin.cnf`的混淆登录路径文件中。在 Windows 上，文件位置是`%APPDATA%\MySQL`目录，在非 Windows 系统上是当前用户的主目录。稍后，MySQL 客户端程序可以读取该文件以获取连接到 MySQL 服务器的身份验证凭据。

`.mylogin.cnf`登录路径文件的未混淆格式由选项组组成，类似于其他选项文件。`.mylogin.cnf`中的每个选项组称为“登录路径”，这是一个仅允许特定选项的组：`host`、`user`、`password`、`port`和`socket`。将登录路径选项组视为一组指定连接到哪个 MySQL 服务器以及以哪个帐户进行身份验证的选项。以下是一个未混淆的示例：

```sql
[client]
user = mydefaultname
password = mydefaultpass
host = 127.0.0.1
[mypath]
user = myothername
password = myotherpass
host = localhost
```

当您调用客户端程序连接到服务器时，客户端将与其他选项文件一起使用`.mylogin.cnf`。它的优先级高于其他选项文件，但低于在客户端命令行上明确指定的选项。有关使用选项文件的顺序，请参阅第 6.2.2.2 节“使用选项文件”。

要指定替代的登录路径文件名，请设置`MYSQL_TEST_LOGIN_FILE`环境变量。此变量被**mysql_config_editor**、标准 MySQL 客户端（**mysql**、**mysqladmin**等）以及**mysql-test-run.pl**测试实用程序识别。

程序使用登录路径文件中的组如下：

+   **mysql_config_editor** 默认情况下在`client`登录路径上运行，如果您未指定`--login-path=*`name`*`选项来明确指示要使用哪个登录路径。

+   如果没有`--login-path`选项，客户端程序将从登录路径文件中读取与从其他选项文件中读取的相同选项组。考虑以下命令：

    ```sql
    mysql
    ```

    默认情况下，**mysql**客户端从其他选项文件中读取`[client]`和`[mysql]`组，因此也从登录路径文件中读取它们。

+   使用`--login-path`选项时，客户端程序还会从登录路径文件中读取指定的登录路径。从其他选项文件中读取的选项组保持不变。考虑以下命令：

    ```sql
    mysql --login-path=mypath
    ```

    **mysql**客户端从其他选项文件中读取`[client]`和`[mysql]`，以及登录路径文件中的`[client]`、`[mysql]`和`[mypath]`。

+   即使使用`--no-defaults`选项，客户端程序也会读取登录路径文件，除非设置了`--no-login-paths`。这样即使存在`--no-defaults`，也可以以比在命令行上更安全的方式指定密码。

**mysql_config_editor**对`.mylogin.cnf`文件进行混淆，以防止以明文形式读取，当客户端程序解密后仅在内存中使用其内容。这样，密码可以以非明文格式存储在文件中，并在以后使用而无需在命令行或环境变量中暴露。**mysql_config_editor**提供了一个`print`命令来显示登录路径文件的内容，但即使在这种情况下，密码值也会被掩盖，以便其他用户永远看不到它们。

**mysql_config_editor**使用的混淆方式防止密码以明文形式出现在`.mylogin.cnf`中，并通过防止密码意外暴露提供了一定的安全性。例如，如果在屏幕上显示一个常规的未混淆的`my.cnf`选项文件，其中包含的任何密码都会对任何人可见。但对于`.mylogin.cnf`，情况并非如此，但所使用的混淆方式不太可能阻止一个有决心的攻击者，你不应该认为它是无法破解的。一个能够获得系统管理权限以访问你的文件的用户可以通过一些努力来解密`.mylogin.cnf`文件。

登录路径文件必须对当前用户可读可写，对其他用户不可访问。否则，**mysql_config_editor**会忽略它，客户端程序也不会使用它。

像这样调用**mysql_config_editor**：

```sql
mysql_config_editor [*program_options*] *command* [*command_options*]
```

如果登录路径文件不存在，**mysql_config_editor**会创建它。

命令参数如下所示：

+   *`program_options`* 包括一般的**mysql_config_editor**选项。

+   `command` 指示在`.mylogin.cnf`登录路径文件上执行的操作。例如，`set`将登录路径写入文件，`remove`删除登录路径，`print`显示登录路径内容。

+   *`command_options`* 表示特定于命令的任何其他选项，例如登录路径名称和在登录路径中使用的值。

命令名称在程序参数集中的位置很重要。例如，这些命令行具有相同的参数，但产生不同的结果：

```sql
mysql_config_editor --help set
mysql_config_editor set --help
```

第一条命令行显示一条一般的**mysql_config_editor**帮助消息，并忽略`set`命令。第二条命令行显示特定于`set`命令的帮助消息。

假设您想要建立一个定义默认连接参数的`client`登录路径，以及一个名为`remote`的额外登录路径，用于连接到主机`remote.example.com`���MySQL 服务器。您想要如下登录：

+   默认情况下，使用用户名和密码`localuser`和`localpass`连接到本地服务器

+   使用用户名和密码`remoteuser`和`remotepass`连接到远程服务器

要在`.mylogin.cnf`文件中设置登录路径，请使用以下`set`命令。每个命令都要单独输入一行，并在提示时输入适当的密码：

```sql
$> mysql_config_editor set --login-path=client
         --host=localhost --user=localuser --password
Enter password: *enter password "localpass" here*
$> mysql_config_editor set --login-path=remote
         --host=remote.example.com --user=remoteuser --password
Enter password: *enter password "remotepass" here*
```

**mysql_config_editor**默认使用`client`登录路径，因此第一个命令可以省略`--login-path=client`选项而不改变其效果。

要查看**mysql_config_editor**写入`.mylogin.cnf`文件的内容，请使用`print`命令：

```sql
$> mysql_config_editor print --all
[client]
user = localuser
password = *****
host = localhost
[remote]
user = remoteuser
password = *****
host = remote.example.com
```

`print` 命令将每个登录路径显示为一组以方括号中的登录路径名称开头的行，后跟登录路径的选项值。密码值被屏蔽，不会显示为明文。

如果您不指定`--all`以显示所有登录路径或`--login-path=*`name`*`以显示命名的登录路径，则`print`命令默认显示`client`登录路径（如果有）。

如前面的示例所示，登录路径文件可以包含多个登录路径。通过这种方式，**mysql_config_editor**可以轻松设置多个连接到不同 MySQL 服务器的“个性”，或者使用不同帐户连接到给定服务器。稍后可以通过名称使用`--login-path`选项选择其中任何一个连接到客户端程序。例如，要连接到远程服务器，请使用此命令：

```sql
mysql --login-path=remote
```

在这里，**mysql** 从其他选项文件中读取 `[client]` 和 `[mysql]` 选项组，从登录路径文件中读取 `[client]`, `[mysql]` 和 `[remote]` 选项组。

要连接到本地服务器，请使用以下命令：

```sql
mysql --login-path=client
```

因为 **mysql** 默认读取 `client` 和 `mysql` 登录路径，所以在这种情况下 `--login-path` 选项不会添加任何内容。该命令等效于以下命令：

```sql
mysql
```

从登录路径文件中读取的选项优先于从其他选项文件中读取的选项。在登录路径文件中后出现的登录路径组的选项优先于文件中较早出现的组的选项。

**mysql_config_editor** 按照创建它们的顺序将登录路径添加到登录路径文件中，因此应该首先创建更通用的登录路径，然后再创建更具体的路径。如果需要在文件中移动登录路径，可以将其删除，然后重新创建以将其添加到末尾。例如，`client` 登录路径更通用，因为所有客户端程序都会读取它，而 `mysqldump` 登录路径仅被 **mysqldump** 读取。后面指定的选项会覆盖先前指定的选项，因此按照 `client`, `mysqldump` 的顺序放置登录路径使得 **mysqldump** 特定的选项能够覆盖 `client` 的选项。

当使用 `set` 命令与 **mysql_config_editor** 创建登录路径时，无需指定所有可能的选项值（主机名、用户名、密码、端口、套接字）。只有给定的值会被写入路径。稍后需要的任何缺失值可以在调用客户端路径连接到 MySQL 服务器时指定，无论是在其他选项文件中还是在命令行上。在命令行上指定的任何选项会覆盖登录路径文件或其他选项文件中指定的选项。例如，如果 `remote` 登录路径中的凭据也适用于主机 `remote2.example.com`，可以像这样连接到该主机的服务器：

```sql
mysql --login-path=remote --host=remote2.example.com
```

#### mysql_config_editor 通用选项

**mysql_config_editor** 支持以下通用选项，可用于在命令行上任何命令名称之前使用。有关特定命令选项的描述，请参见 mysql_config_editor 命令和特定命令选项。

**表 6.21 mysql_config_editor 通用选项**

| 选项名称 | 描述 |
| --- | --- |
| --debug | 写入调试日志 |
| --help | 显示帮助消息并退出 |
| --verbose | 详细模式 |
| --version | 显示版本信息并退出 |

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示通用帮助消息并退出。

    要查看特定于命令的帮助消息，请按照以下方式调用**mysql_config_editor**，其中*`command`*是除`help`之外的命令：

    ```sql
    mysql_config_editor *command* --help
    ```

+   [`--debug[=*`debug_options`*]`](mysql-config-editor.html#option_mysql_config_editor_debug), `-# *`debug_options`*`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:o` |

    写入调试日志。典型的*`debug_options`*字符串是`d:t:o,*`file_name`*`。默认值为`d:t:o,/tmp/mysql_config_editor.trace`。

    此选项仅在使用`WITH_DEBUG`构建 MySQL 时可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。打印有关程序执行的更多信息。如果操作没有您期望的效果，此选项可能有助于诊断问题。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

#### mysql_config_editor 命令和特定于命令的选项

本节描述了允许的**mysql_config_editor**命令，以及每个命令后面允许的特定于命令的选项。

此外，**mysql_config_editor**支持可用于在任何命令之前使用的通用选项。有关这些选项的描述，请参阅 mysql_config_editor 通用选项。

**mysql_config_editor**支持以下命令：

+   `help`

    显示通用帮助消息并退出。此命令不接受后续选项。

    要查看特定于命令的帮助消息，请按照以下方式调用**mysql_config_editor**，其中*`command`*是除`help`之外的命令：

    ```sql
    mysql_config_editor *command* --help
    ```

+   `print [*`options`*]`

    以未混淆形式打印登录路径文件的内容，密码显示为`*****`。

    如果没有指定登录路径，则默认登录路径名称为`client`。如果同时给出`--all`和`--login-path`，则`--all`优先。

    `print`命令允许在命令名称后面使用以下选项：

    +   `--help`, `-?`

        显示`print`命令的帮助信息并退出。

        要查看通用帮助信息，请使用**mysql_config_editor --help**。

    +   `--all`

        打印登录路径文件中所有登录路径的内容。

    +   `--login-path=*`name`*`, `-G *`name`*`

        打印指定名称登录路径的内容。

+   `remove [*`options`*]`

    从登录路径文件中移除登录路径，或通过移除选项来修改登录路径。

    此命令仅移除登录路径中指定的`--host`、`--password`、`--port`、`--socket`和`--user`选项。如果没有给出这些选项中的任何一个，`remove`将移除整个登录路径。例如，此命令仅从`mypath`登录路径中移除`user`选项，而不是整个`mypath`登录路径：

    ```sql
    mysql_config_editor remove --login-path=mypath --user
    ```

    此命令将移除整个`mypath`登录路径：

    ```sql
    mysql_config_editor remove --login-path=mypath
    ```

    `remove`命令允许在命令名称后面使用以下选项：

    +   `--help`, `-?`

        显示`remove`命令的帮助信息并退出。

        要查看通用帮助信息，请使用**mysql_config_editor --help**。

    +   `--host`, `-h`

        从登录路径中移除主机名。

    +   `--login-path=*`name`*`, `-G *`name`*`

        要移除或修改的登录路径。如果未给出此选项，则默认登录路径名称为`client`。

    +   `--password`, `-p`

        从登录路径中移除密码。

    +   `--port`, `-P`

        从登录路径中移除 TCP/IP 端口号。

    +   `--socket`, `-S`

        从登录路径中移除 Unix 套接字文件名。

    +   `--user`, `-u`

        从登录路径中移除用户名。

    +   `--warn`, `-w`

        如果尝试移除默认登录路径（`client`）且未指定`--login-path=client`，则警告并提示用户确认。此选项默认启用；使用`--skip-warn`来禁用。

+   `reset [*`options`*]`

    清空登录路径文件的内容。

    `reset`命令允许在命令名称后面使用以下选项：

    +   `--help`, `-?`

        显示`reset`命令的帮助信息并退出。

        要查��通用帮助信息，请使用**mysql_config_editor --help**。

+   `set [*`options`*]`

    将登录路径写入登录路径文件。

    此命令仅将`--host`、`--password`、`--port`、`--socket`和`--user`选项指定的选项写入登录路径。如果没有给出这些选项中的任何一个，**mysql_config_editor**将登录路径写入为空组。

    `set`命令允许在命令名称后面使用以下选项：

    +   `--help`, `-?`

        显示`set`命令的帮助消息并退出。

        要查看一般帮助消息，请使用**mysql_config_editor --help**。

    +   `--host=*`host_name`*`, `-h *`host_name`*`

        写入登录路径的主机名。

    +   `--login-path=*`name`*`, `-G *`name`*`

        要创建的登录路径。如果未提供此选项，则默认登录路径名称为`client`。

    +   `--password`, `-p`

        提示输入要写入登录路径的密码。在**mysql_config_editor**显示提示后，输入密码并按 Enter 键。为防止其他用户看到密码，**mysql_config_editor**不会回显密码。

        要指定空密码，请在密码提示处按 Enter 键。写入登录路径文件的结果包括如下一行：

        ```sql
        password =
        ```

    +   `--port=*`port_num`*`, `-P *`port_num`*`

        写入登录路径的 TCP/IP 端口号。

    +   `--socket=*`file_name`*`, `-S *`file_name`*`

        写入登录路径的 Unix 套接字文件名。

    +   `--user=*`user_name`*`, `-u *`user_name`*`

        要写入登录路径的用户名。

    +   `--warn`, `-w`

        如果命令尝试覆盖现有的登录路径，则警告并提示用户确认。此选项默认启用；使用`--skip-warn`来禁用它。
