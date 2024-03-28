# 8.2.22 连接到 MySQL 时出现问题的故障排除

> 原文：[`dev.mysql.com/doc/refman/8.0/en/problems-connecting.html`](https://dev.mysql.com/doc/refman/8.0/en/problems-connecting.html)

如果尝试连接到 MySQL 服务器时遇到问题，以下内容描述了您可以采取的一些措施来纠正问题。

+   确保服务器正在运行。如果没有运行，客户端无法连接到它。例如，如果尝试连接到服务器失败，并显示以下消息之一，可能的原因之一是服务器未运行：

    ```sql
    $> mysql
    ERROR 2003: Can't connect to MySQL server on '*host_name*' (111)
    $> mysql
    ERROR 2002: Can't connect to local MySQL server through socket
    '/tmp/mysql.sock' (111)
    ```

+   可能是服务器正在运行，但您尝试使用与服务器监听的 TCP/IP 端口、命名管道或 Unix 套接字文件不同的端口。当您调用客户端程序时，指定`--port`选项以指示正确的端口号，或者指定`--socket`选项以指示正确的命名管道或 Unix 套接字文件。要找出套接字文件的位置，您可以使用以下命令：

    ```sql
    $> netstat -ln | grep mysql
    ```

+   确保服务器未配置为忽略网络连接，或者（如果您尝试远程连接）未配置为仅在其网络接口上本地监听。如果服务器启用了`skip_networking`系统变量，将不接受 TCP/IP 连接。如果服务器启用了`bind_address`系统变量设置为`127.0.0.1`，它只在回环接口上本地监听 TCP/IP 连接，不接受远程连接。

+   检查确保没有防火墙阻止访问 MySQL。您的防火墙可能根据正在执行的应用程序或 MySQL 用于通信的端口号（默认为 3306）进行配置。在 Linux 或 Unix 下，请检查您的 IP 表（或类似）配置，确保端口未被阻止。在 Windows 下，诸如 ZoneAlarm 或 Windows 防火墙等应用程序可能需要配置不要阻止 MySQL 端口。

+   授予权限表必须正确设置，以便服务器可以使用它们进行访问控制。对于某些发行版类型（例如 Windows 上的二进制发行版，或 Linux 上的 RPM 和 DEB 发行版），安装过程会初始化 MySQL 数据目录，包括包含授予权限表的`mysql`系统数据库。对于不执行此操作的发行版，您必须手动初始化数据目录。有关详细信息，请参见第 2.9 节，“安装后设置和测试”。

    要确定是否需要初始化授权表，请查找数据目录下是否有一个`mysql`目录。（数据目录通常命名为`data`或`var`，位于您的 MySQL 安装目录下。）确保在`mysql`数据库目录中有一个名为`user.MYD`的文件。如果没有，请初始化数据目录。完成后启动服务器，您应该能够连接到服务器。

+   在全新安装后，如果尝试以不使用密码的方式作为`root`登录到服务器，则可能会收到以下错���消息。

    ```sql
    $> mysql -u root 
    ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
    ```

    这意味着在安装过程中已经分配了一个 root 密码，并且必须提供该密码。请查看 Section 2.9.4, “Securing the Initial MySQL Account”，了解密码可能是如何分配的，以及在某些情况下如何找到它。如果需要重置 root 密码，请查看 Section B.3.3.2, “How to Reset the Root Password”中的说明。找到或重置密码后，请再次以`root`身份登录，使用`--password`（或`-p`）选项：

    ```sql
    $> mysql -u root -p
    Enter password:
    ```

    然而，如果您使用**mysqld --initialize-insecure**初始化 MySQL（详见 Section 2.9.1, “Initializing the Data Directory”），服务器将允许您以`root`身份连接而无需密码。这是一个安全风险，因此您应该为`root`账户设置密码；请参阅 Section 2.9.4, “Securing the Initial MySQL Account”获取指导。

+   如果您将现有的 MySQL 安装升级到新版本，您是否执行了 MySQL 升级过程？如果没有，请执行。当添加新功能时，授权表的结构偶尔会发生变化，因此在升级后，您应始终确保您的表具有当前结构。有关说明，请参见 Chapter 3, *Upgrading MySQL*。

+   如果客户端程序在尝试连接时收到以下错误消息，则表示服务器期望密码的格式比客户端能够生成的格式更新：

    ```sql
    $> mysql
    Client does not support authentication protocol requested
    by server; consider upgrading MySQL client
    ```

+   客户端程序使用在选项文件或环境变量中指定的连接参数。如果客户端程序在没有在命令行上指定默认连接参数时似乎发送了不正确的连接参数，请检查任何适用的选项文件和您的环境。例如，如果在运行一个没有任何选项的客户端时收到`Access denied`，请确保您没有在任何选项文件中指定旧密码！

    您可以通过使用`--no-defaults`选项调用客户端程序来抑制选项文件的使用。例如：

    ```sql
    $> mysqladmin --no-defaults -u root version
    ```

    客户端使用的选项文件列在第 6.2.2.2 节，“使用选项文件”中。环境变量列在第 6.9 节，“环境变量”中。

+   如果收到以下错误，则表示您正在使用不正确的`root`密码：

    ```sql
    $> mysqladmin -u root -p*xxxx* ver
    Access denied for user 'root'@'localhost' (using password: YES)
    ```

    即使您没有指定密码，但出现上述错误，这意味着某个选项文件中列出了不正确的密码。尝试如前一项所述的`--no-defaults`选项。

    更改密码的信息，请参阅第 8.2.14 节，“分配帐户密码”。

    如果您忘记或丢失了`root`密码，请参阅第 B.3.3.2 节，“如何重置根密码”。

+   `localhost`是您的本地主机名的同义词，也是客户端尝试连接的默认主机，如果您未明确指定主机。

    您可以使用`--host=127.0.0.1`选项显式命名服务器主机。这将导致与本地**mysqld**服务器的 TCP/IP 连接。您还可以通过指定使用实际主机名的`--host`选项来使用 TCP/IP。在这种情况下，即使您在同一主机上运行客户端程序，主机名也必须在服务器主机上的`user`表行中指定。

+   “拒绝访问”错误消息告诉您尝试登录的用户、您尝试连接的客户端主机以及是否使用了密码。通常，您应该在`user`表中有一行与错误消息中给定的主机名和用户名完全匹配。例如，如果收到包含`using password: NO`的错误消息，则表示您尝试在没有密码的情况下登录。

+   如果尝试使用`mysql -u *`user_name`*`连接到数据库时出现“拒绝访问”错误，则可能存在`user`表的问题。通过执行`mysql -u root mysql`并发出以下 SQL 语句来检查：

    ```sql
    SELECT * FROM user;
    ```

    结果应包括`Host`和`User`列与您的客户端主机名和 MySQL 用户名匹配的行。

+   如果在尝试从 MySQL 服务器运行的主机之外的主机连接时出现以下错误，则表示`user`表中没有`Host`值与客户端主机匹配的行：

    ```sql
    Host ... is not allowed to connect to this MySQL server
    ```

    您可以通过为尝试连接时使用的客户端主机名和用户名设置一个帐户来解决此问题。

    如果您不知道正在连接的机器的 IP 地址或主机名，应在`user`表中将`Host`列值设置为`'%'`。尝试从客户端机器连接后，使用`SELECT USER()`查询查看您实际连接的方式。然后将`user`表中的`'%'`更改为日志中显示的实际主机名。否则，您的系统将因为允许给定用户名的任何主机连接而变得不安全。

    在 Linux 上，出现此错误的另一个原因可能是您使用的二进制 MySQL 版本是使用与您使用的`glibc`库不同版本编译的。在这种情况下，您应该升级操作系统或`glibc`，或者下载 MySQL 版本的源代码分发并自行编译。源 RPM 通常很容易编译和安装，因此这不是一个大问题。

+   如果尝试连接时指定了主机名，但出现未显示主机名或为 IP 地址的错误消息，则表示 MySQL 服务器在尝试将客户端主机的 IP 地址解析为名称时出现错误：

    ```sql
    $> mysqladmin -u root -p*xxxx* -h *some_hostname* ver
    Access denied for user 'root'@'' (using password: YES)
    ```

    如果尝试以`root`身份连接并收到以下错误，则表示您在`user`表中没有一个`User`列值为`'root'`的行，而**mysqld**无法解析客户端的主机名：

    ```sql
    Access denied for user ''@'unknown'
    ```

    这些错误指示 DNS 问题。要修复它，请执行**mysqladmin flush-hosts**以重置内部 DNS 主机缓存。请参见第 7.1.12.3 节，“DNS 查找和主机缓存”。

    一些永久解决方案包括：

    +   确定您的 DNS 服务器出了什么问题并加以修复。

    +   在 MySQL 授权表中指定 IP 地址而不是主机名。

    +   在 Unix 的`/etc/hosts`或 Windows 的`\windows\hosts`中为客户端机器名称添加条目。

    +   使用启用了`skip_name_resolve`系统变量启动**mysqld**。

    +   使用`--skip-host-cache`选项启动**mysqld**。

    +   在 Unix 上，如果服务器和客户端在同一台机器上运行，请连接到`localhost`。对于连接到`localhost`的连接，MySQL 程序尝试使用 Unix 套接字文件连接到本地服务器，除非指定了连接参数以确保客户端进行 TCP/IP 连接。有关更多信息，请参见第 6.2.4 节，“使用命令选项连接到 MySQL 服务器”。

    +   在 Windows 上，如果您在同一台机器上运行服务器和客户端，并且服务器支持命名管道连接，请连接到主机名`.`（句点）。连接到`.`使用命名管道而不是 TCP/IP。

+   如果`mysql -u root`有效，但`mysql -h *your_hostname* -u root`导致`拒绝访问`（其中*your_hostname*是本地主机的实际主机名），则可能在`user`表中没有为您的主机指定正确的名称。这里一个常见的问题是`user`表行中的`Host`值指定了一个未经验证的主机名，但您系统的名称解析例程返回了一个完全限定的域名（或反之）。例如，如果`user`表中有一个带有主机`'pluto'`的行，但您的 DNS 告诉 MySQL 您的主机名是`'pluto.example.com'`，那么该行将无效。尝试向`user`表中添加一行，其中包含您主机的 IP 地址作为`Host`列值。（或者，您可以向`user`表中添加一个包含通配符的`Host`值的行（例如，`'pluto.%'`）。但是，使用以`%`结尾的`Host`值是*不安全*的，*不*建议使用！）

+   如果`mysql -u *user_name*`有效，但`mysql -u *user_name* *some_db*`无效，则您尚未为名为*some_db*的数据库授予给定用户的访问权限。

+   如果在服务器主机上执行`mysql -u *user_name*`有效，但在远程客户端主机上执行`mysql -h *host_name* -u *user_name*`无效，则您尚未为给定用户从远程主机启用对服务器的访问。

+   如果你无法弄清楚为什么会出现`拒绝访问`的情况，请从`user`表中删除所有`Host`值包含通配符（包含`'%'`或`'_'`字符）的行。一个非常常见的错误是插入一个新行，其中`Host`=`'%'`和`User`=`'*some_user*'`，认为这样可以允许你指定`localhost`从同一台机器连接。这样做不起作用的原因是默认权限包括一行`Host`=`'localhost'`和`User`=`''`。因为该行具有比`'%'`更具体的`Host`值`'localhost'`，所以在从`localhost`连接时会优先使用该行而不是新行！正确的做法是插入第二行，其中`Host`=`'localhost'`和`User`=`'*some_user*'`，或者删除`Host`=`'localhost'`和`User`=`''`的行。删除行后，请记得发出`FLUSH PRIVILEGES`语句以重新加载授权表。另请参阅 Section 8.2.6，“访问控制，阶段 1：连接验证”。

+   如果您能够连接到 MySQL 服务器，但在执行 `SELECT ... INTO OUTFILE` 或 `LOAD DATA` 语句时收到 `Access denied` 消息，那么您在 `user` 表中的行没有启用 `FILE` 权限。

+   如果您直接更改授权表（例如，使用 `INSERT`、`UPDATE` 或 `DELETE` 语句），而您的更改似乎被忽略，请记住您必须执行 `FLUSH PRIVILEGES` 语句或 **mysqladmin flush-privileges** 命令，以使服务器重新加载权限表。否则，您的更改在下次服务器重启之前不会生效。请记住，在使用 `UPDATE` 语句更改 `root` 密码后，直到刷新权限之前，您不需要指定新密码，因为服务器在那时还不知道您已更改密码。

+   如果您在会话中间发现权限似乎已更改，可能是 MySQL 管理员已更改了它们。重新加载授权表会影响新的客户端连接，但也会影响现有连接，如 第 8.2.13 节，“权限更改何时生效” 所示。

+   如果您在 Perl、PHP、Python 或 ODBC 程序中遇到访问问题，请尝试使用 `mysql -u *`user_name`* *`db_name`*` 或 `mysql -u *`user_name`* -p*`password`* *`db_name`*` 连接到服务器。如果您能够使用 **mysql** 客户端连接，问题可能出在您的程序上，而不是访问权限上。（`-p` 和密码之间没有空格；您也可以使用 `--password=*`password`*` 语法来指定密码。如果您使用 `-p` 或 `--password` 选项而没有密码值，MySQL 会提示您输入密码。）

+   为了测试目的，使用`--skip-grant-tables`选项启动**mysqld**服务器。然后您可以更改 MySQL 授权表，并使用`SHOW GRANTS`语句检查您的修改是否产生了预期效果。当您满意您的更改时，执行**mysqladmin flush-privileges**告诉**mysqld**服务器重新加载权限。这使您可以开始使用新的授权表内容，而无需停止和重新启动服务器。

+   如果一切都失败了，请使用调试选项（例如，`--debug=d,general,query`）启动**mysqld**服务器。这将打印有关尝试连接的主机和用户信息，以及有关每个发出的命令的信息。请参阅 Section 7.9.4, “The DBUG Package”。

+   如果您在 MySQL 授权表上遇到任何其他问题，并在[MySQL 社区 Slack](https://mysqlcommunity.slack.com/)上提问，请始终提供 MySQL 授权表的转储。您可以使用**mysqldump mysql**命令转储表格。要提交错误报告，请参阅 Section 1.5, “How to Report Bugs or Problems”中的说明。在某些情况下，您可能需要使用`--skip-grant-tables`重新启动**mysqld**以运行**mysqldump**。
