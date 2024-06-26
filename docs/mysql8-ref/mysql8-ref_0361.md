# 8.2.1 账户用户名和密码

> 原文：[`dev.mysql.com/doc/refman/8.0/en/user-names.html`](https://dev.mysql.com/doc/refman/8.0/en/user-names.html)

MySQL 将账户存储在`mysql`系统数据库的`user`表中。账户根据用户名称和用户可以连接到服务器的客户端主机或主机来定义。有关`user`表中账户表示的信息，请参见 第 8.2.3 节，“授权表”。

一个账户可能还有身份验证凭据，比如密码。这些凭据由账户身份验证插件处理。MySQL 支持多种身份验证插件。其中一些使用内置身份验证方法，而其他一些则启用使用外部身份验证方法进行身份验证。参见 第 8.2.17 节，“可插拔身份验证”。

MySQL 和您的操作系统在使用用户名称和密码方面有几个区别：

+   用户名，用于 MySQL 身份验证目的，与 Windows 或 Unix 中用于登录的用户名无关。在 Unix 上，默认情况下，大多数 MySQL 客户端尝试使用当前 Unix 用户名作为 MySQL 用户名登录，但这仅仅是为了方便。默认设置可以轻松覆盖，因为客户端程序允许使用`-u`或`--user`选项指定任何用户名。这意味着任何人都可以尝试使用任何用户名连接到服务器，因此除非所有 MySQL 账户都有密码，否则无法以任何方式使数据库安全。任何指定了没有密码的账户的用户名的人都可以成功连接到服务器。

+   MySQL 用户名称最长可达 32 个字符。操作系统用户名称可能具有不同的最大长度。

    警告

    MySQL 用户名称长度限制是硬编码在 MySQL 服务器和客户端中的，试图通过修改`mysql`数据库中表的定义来规避它*不起作用*。

    除了通过第三章，“升级 MySQL”中描述的过程之外，您绝对不应以任何方式更改`mysql`数据库中表的结构。试图以任何其他方式重新定义 MySQL 系统表会导致未定义和不受支持的行为。服务器可以忽略由于这些修改而变得畸形的行。

+   为使用内置身份验证方法的账户验证客户端连接，服务器使用存储在`user`表中的密码。这些密码与用于登录操作系统的密码不同。您用于登录 Windows 或 Unix 机器的“外部”密码与您用于访问该机器上的 MySQL 服务器的密码之间没有必要的联系。

    如果服务器使用其他插件对客户端进行身份验证，插件实现的身份验证方法可能会或可能不会使用存储在`user`表中的密码。在这种情况下，可能还会使用外部密码进行 MySQL 服务器的身份验证。

+   存储在`user`表中的密码使用特定于插件的算法进行加密。

+   如果用户名和密码只包含 ASCII 字符，则无论字符集设置如何，都可以连接到服务器。要在用户名或密码包含非 ASCII 字符时启用连接，客户端应用程序应使用`MYSQL_SET_CHARSET_NAME`选项和适当的字符集名称作为参数调用`mysql_options()` C API 函数。这会导致使用指定的字符集进行身份验证。否则，除非服务器默认字符集与身份验证默认值中的编码相同，否则身份验证将失败。

    标准的 MySQL 客户端程序支持一个`--default-character-set`选项，会导致调用`mysql_options()`如上所述。此外，支持字符集自动检测，如第 12.4 节，“连接字符集和校对规则”所述。对于不基于 C API 的连接器，可能会提供类似于`mysql_options()`的等效选项供使用。请查阅连接器文档。

    前述注意事项不适用于`ucs2`、`utf16`和`utf32`，这些字符集不被允许作为客户端字符集。

MySQL 安装过程会使用初始`root`账户填充授权表，如第 2.9.4 节，“保护初始 MySQL 账户”所述，该节还讨论了如何为其分配密码。之后，通常使用`CREATE USER`、`DROP USER`、`GRANT`和`REVOKE`等语句设置、修改和删除 MySQL 账户。参见第 8.2.8 节，“添加账户、分配权限和删除账户”，以及第 15.7.1 节，“账户管理语句”。

要使用命令行客户端连接到 MySQL 服务器，根据要使用的账户，指定用户名和密码选项如下所示：

```sql
$> mysql --user=finley --password *db_name*
```

如果您喜欢简短选项，命令如下：

```sql
$> mysql -u finley -p *db_name*
```

如果在命令行上省略了`--password`或`-p`选项后面的密码值（如上所示），客户端会提示输入密码。或者，密码可以在命令行上指定：

```sql
$> mysql --user=finley --password=*password* *db_name*
$> mysql -u finley -p*password* *db_name*
```

如果使用`-p`选项，则`-p`和后面的密码值之间*不能有空格*。

在命令行上指定密码应被视为不安全。请参见第 8.1.2.1 节，“密码安全的最终用户指南”。为了避免在命令行上输入密码，可以使用选项文件或登录路径文件。请参见第 6.2.2.2 节，“使用选项文件”，以及第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

有关指定用户名、密码和其他连接参数的附加信息，请参见第 6.2.4 节，“使用命令选项连接到 MySQL 服务器”。
