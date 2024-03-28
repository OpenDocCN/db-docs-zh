# 8.1.3 使 MySQL 免受攻击者攻击

> 译文：[`dev.mysql.com/doc/refman/8.0/en/security-against-attack.html`](https://dev.mysql.com/doc/refman/8.0/en/security-against-attack.html)

当连接到 MySQL 服务器时，应该使用密码。密码不会以明文形式通过连接传输。

所有其他信息都以文本形式传输，可以被能够监视连接的任何人读取。如果客户端和服务器之间的连接经过不受信任的网络，并且您对此感到担忧，您可以使用压缩协议使流量更加难以解密。您还可以使用 MySQL 的内部 SSL 支持使连接更加安全。请参见 Section 8.3, “Using Encrypted Connections”。或者，使用 SSH 在 MySQL 服务器和 MySQL 客户端之间建立加密的 TCP/IP 连接。您可以在[`www.openssh.org/`](http://www.openssh.org/)找到一个开源 SSH 客户端，并在[`en.wikipedia.org/wiki/Comparison_of_SSH_clients`](http://en.wikipedia.org/wiki/Comparison_of_SSH_clients)找到开源和商业 SSH 客户端的比较。

要使 MySQL 系统安全，您应该强烈考虑以下建议：

+   要求所有 MySQL 账户都有密码。客户端程序不一定知道运行它的人的身份。对于客户端/服务器应用程序来说，用户可以向客户端程序指定任何用户名是很常见的。例如，任何人都可以使用**mysql**程序连接为任何其他人，只需通过`mysql -u *`other_user`* *`db_name`*`来调用，如果*`other_user`*没有密码。如果所有账户都有密码，使用另一个用户的账户连接就变得更加困难。

    有关设置密码方法的讨论，请参见 Section 8.2.14, “Assigning Account Passwords”。

+   确保在数据库目录中具有读取或写入权限的唯一 Unix 用户帐户是用于运行**mysqld**的帐户。

+   永远不要将 MySQL 服务器作为 Unix `root`用户运行。这是极其危险的，因为任何具有`FILE`权限的用户都可以使服务器以`root`身份创建文件（例如，`~root/.bashrc`）。为了防止这种情况，**mysqld**拒绝以`root`身份运行，除非明确使用`--user=root`选项指定。

    **mysqld**可以（而且应该）作为一个普通的、非特权用户运行。您可以创建一个名为`mysql`的单独的 Unix 帐户，以使一切更加安全。仅使用此帐户来管理 MySQL。要将**mysqld**作为不同的 Unix 用户启动，请在指定服务器选项的`my.cnf`选项文件的`[mysqld]`组中添加一个指定用户名称的`user`选项。例如：

    ```sql
    [mysqld]
    user=mysql
    ```

    这将导致服务器作为指定用户启动，无论您是手动启动还是使用**mysqld_safe**或**mysql.server**启动。有关更多详细信息，请参见 Section 8.1.5，“如何将 MySQL 作为普通用户运行”。

    将**mysqld**作为`root`以外的 Unix 用户运行并不意味着您需要更改`user`表中的`root`用户名。*MySQL 帐户的用户名与 Unix 帐户的用户名无关*。

+   不要将`FILE`权限授予非管理员用户。拥有此权限的任何用户都可以以**mysqld**守护程序的权限在文件系统中的任何位置写入文件。这包括包含实现权限表的文件的服务器数据目录。为了使具有`FILE`权限的操作更安全一些，使用`SELECT ... INTO OUTFILE`生成的文件不会覆盖现有文件，并且可以被所有人写入。

    `FILE`权限也可以用于读取任何对所有用户可读或对服务器运行的 Unix 用户可访问的文件。有了这个权限，您可以将任何文件读入数据库表中。例如，可以通过使用`LOAD DATA`将`/etc/passwd`加载到表中，然后可以使用`SELECT`显示。

    为了限制可以读取和写入文件的位置，将`secure_file_priv`系统设置为特定目录。请参见 Section 7.1.8，“服务器系统变量”。

+   对二进制日志文件和中继日志文件进行加密。加密有助于保护这些文件和其中可能包含的敏感数据，防止外部攻击者滥用，也防止存储它们的操作系统用户未经授权查看。您可以通过将`binlog_encryption`系统变量设置为`ON`来在 MySQL 服务器上启用加密。有关更多信息，请参见第 19.3.2 节，“加密二进制日志文件和中继日志文件”。

+   不要向非管理员用户授予`PROCESS`或`SUPER`权限。**mysqladmin processlist**和`SHOW PROCESSLIST`的输出显示当前正在执行的任何语句的文本，因此任何被允许查看服务器进程列表的用户可能能够看到其他用户发出的语句。

    **mysqld**为具有`CONNECTION_ADMIN`或`SUPER`权限的用户保留了额外的连接，以便 MySQL `root`用户可以登录并检查服务器活动，即使所有正常连接都在使用中。

    `SUPER`权限可用于终止客户端连接，通过更改系统变量的值改变服务器操作，并控制复制服务器。

+   不要允许使用符号链接到表格。 （可以使用`--skip-symbolic-links`选项禁用此功能。）如果您以`root`身份运行**mysqld**，那么任何具有对服务器数据目录的写访问权限的人都可以删除系统中的任何文件！请参见第 10.12.2.2 节，“在 Unix 上为 MyISAM 表使用符号链接”。

+   存储过程和视图应该按照第 27.6 节，“存储对象访问控制”中讨论的安全准则编写。

+   如果您不信任您的 DNS，应该在授权表中使用 IP 地址而不是主机名。无论如何，您都应该非常小心地创建包含通配符的主机名值的授权表条目。

+   如果您想限制单个帐户允许的连接数，可以通过在**mysqld**中设置`max_user_connections`变量来实现。`CREATE USER`和`ALTER USER`语句还支持资源控制选项，用于限制帐户允许使用服务器的范围。请参阅 Section 15.7.1.3, “CREATE USER Statement”和 Section 15.7.1.1, “ALTER USER Statement”。

+   如果插件目录对服务器是可写的，用户可能会使用`SELECT ... INTO DUMPFILE`将可执行代码写入目录中的文件。可以通过将`plugin_dir`设置为服务器只读或将`secure_file_priv`设置为`SELECT`可以安全写入的目录来防止这种情况发生。
