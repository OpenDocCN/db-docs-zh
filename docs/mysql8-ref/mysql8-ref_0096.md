# 2.9.4 保护初始 MySQL 帐户

> 原文：[`dev.mysql.com/doc/refman/8.0/en/default-privileges.html`](https://dev.mysql.com/doc/refman/8.0/en/default-privileges.html)

MySQL 安装过程涉及初始化数据目录，包括在`mysql`系统模式中定义 MySQL 帐户的授权表。有关详细信息，请参见第 2.9.1 节，“初始化数据目录”。

本节描述如何为在 MySQL 安装过程中创建的初始`root`帐户分配密码，如果您尚未这样做。

注意

执行本节描述的过程的替代方法：

+   在 Windows 上，您可以在安装过程中使用 MySQL Installer 执行该过程（参见第 2.3.3 节，“Windows 上的 MySQL Installer”）。

+   在所有平台上，MySQL 发行版包括**mysql_secure_installation**，一个命令行实用程序，自动化了大部分保护 MySQL 安装过程的过程。

+   在所有平台上，MySQL Workbench 可用，并提供管理用户帐户的功能（参见第三十三章，*MySQL Workbench*）。

在以下情况下，初始帐户可能已经分配了密码：

+   在 Windows 上，使用 MySQL Installer 执行的安装会让您选择分配密码。

+   使用 macOS 安装程序进行安装会生成一个初始的随机密码，并在对话框中向用户显示。

+   使用 RPM 软件包进行安装会生成一个初始的随机密码，并写入服务器错误日志。

+   使用 Debian 软件包进行安装会让您选择分配密码。

+   对于手动执行数据目录初始化的情况，使用**mysqld --initialize**，**mysqld**会生成一个初始的随机密码，标记为过期，并写入服务器错误日志。参见第 2.9.1 节，“初始化数据目录”。

`mysql.user`授权表定义了初始的 MySQL 用户帐户及其访问权限。MySQL 的安装仅创建一个具有所有权限并可以执行任何操作的`'root'@'localhost'`超级用户帐户。如果`root`帐户没有密码，您的 MySQL 安装是不受保护的：任何人都可以连接到 MySQL 服务器作为`root`，*无需密码*并被授予所有权限。

`'root'@'localhost'`账户还在`mysql.proxies_priv`表中有一行，允许为`''@''`（即所有用户和所有主机）授予`PROXY`权限。这使`root`能够设置代理用户，以及委派其他账户设置代理用户的权限。参见 Section 8.2.19, “Proxy Users”。

要为初始的 MySQL `root`账户分配密码，请使用以下程序。在示例中用你想要使用的密码替换*`root-password`*。

如果服务器未运行，请启动服务器。有关说明，请参见 Section 2.9.2, “Starting the Server”。

初始的`root`账户可能有密码，也可能没有。选择以下适用的任一程序：

+   如果`root`账户存在并且初始的随机密码已过期，请使用该密码连接到服务器作为`root`，然后选择一个新密码。如果数据目录是使用**mysqld --initialize**初始化的，无论是手动还是使用安装程序，在安装操作期间没有指定密码选项。因为密码存在，你必须使用它连接到服务器。但因为密码已过期，你不能将该账户用于除选择新密码之外的任何目的，直到你选择一个新密码。

    1.  如果你不知道初始的随机密码，查看服务器错误日志。

    1.  使用密码连接到服务器作为`root`：

        ```sql
        $> mysql -u root -p
        Enter password: *(enter the random root password here)*
        ```

    1.  选择一个新密码以替换随机密码：

        ```sql
        mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '*root-password*';
        ```

+   如果`root`账户存在但没有密码，请使用无密码连接到服务器作为`root`，然后分配一个密码。如果你使用**mysqld --initialize-insecure**初始化数据目录，则是这种情况。

    1.  使用无密码连接到服务器作为`root`：

        ```sql
        $> mysql -u root --skip-password
        ```

    1.  分配密码：

        ```sql
        mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '*root-password*';
        ```

为`root`账户分配密码后，每次使用该账户连接到服务器时，都必须提供该密码。例如，要使用**mysql**客户端连接到服务器，请使用此命令：

```sql
$> mysql -u root -p
Enter password: *(enter root password here)*
```

要使用**mysqladmin**关闭服务器，请使用此命令：

```sql
$> mysqladmin -u root -p shutdown
Enter password: *(enter root password here)*
```

注意

有关设置密码的更多信息，请参见 Section 8.2.14, “Assigning Account Passwords”。如果设置后忘记了`root`密码，请参见 Section B.3.3.2, “How to Reset the Root Password”。

要设置额外的账户，请参见 Section 8.2.8, “Adding Accounts, Assigning Privileges, and Dropping Accounts”。
