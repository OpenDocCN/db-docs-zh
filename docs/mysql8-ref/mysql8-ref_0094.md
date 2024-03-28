> 原文：[`dev.mysql.com/doc/refman/8.0/en/starting-server-troubleshooting.html`](https://dev.mysql.com/doc/refman/8.0/en/starting-server-troubleshooting.html)

#### 2.9.2.1 启动 MySQL 服务器时出现问题的故障排除

本节提供了有关启动服务器时出现问题的故障排除建议。有关 Windows 系统的其他建议，请参阅第 2.3.5 节，“解决 Microsoft Windows MySQL 服务器安装问题”。

如果您在启动服务器时遇到问题，请尝试以下方法：

+   检查错误日志以查看服务器为何无法启动。日志文件位于数据目录中（在 Windows 上通常为`C:\Program Files\MySQL\MySQL Server 8.0\data`，对于 Unix/Linux 二进制分发为`/usr/local/mysql/data`，对于 Unix/Linux 源码分发为`/usr/local/var`）。在数据目录中查找名称形式为`*`host_name`*.err`和`*`host_name`*.log`的文件，其中*`host_name`*是您服务器主机的名称。然后检查这些文件的最后几行。使用`tail`命令显示它们：

    ```sql
    $> tail *host_name*.err
    $> tail *host_name*.log
    ```

+   指定存储引擎所需的任何特殊选项。您可以创建一个`my.cnf`文件，并为您计划使用的引擎指定启动选项。如果您要使用支持事务表的存储引擎（`InnoDB`，`NDB`），请确保在启动服务器之前将它们配置为您想要的方式。如果您使用`InnoDB`表，请参阅第 17.8 节，“InnoDB 配置”以获取指南，以及第 17.14 节，“InnoDB 启动选项和系统变量”以获取选项语法。

    尽管存储引擎对您省略的选项使用默认值，但 Oracle 建议您查看可用选项，并为任何默认值不适合您安装的选项指定显式值。

+   确保服务器知道在哪里找到数据目录。**mysqld**服务器将此目录用作其当前目录。这是它期望找到数据库并写入日志文件的地方。服务器还会在数据目录中写入 pid（进程 ID）文件。

    默认数据目录位置在服务器编译时被硬编码。要确定默认路径设置是什么，使用带有`--verbose`和`--help`选项调用**mysqld**。如果数据目录位于系统的其他位置，请使用`--datadir`选项指定该位置给**mysqld**或**mysqld_safe**，在命令行或选项文件中。否则，服务器将无法正常工作。作为`--datadir`选项的替代方案，您可以使用`--basedir`指定 MySQL 安装的基本目录的位置给**mysqld**，然后**mysqld**在那里查找`data`目录。

    要检查指定路径选项的效果，请使用这些选项调用**mysqld**，然后跟随`--verbose`和`--help`选项。例如，如果将位置更改为安装**mysqld**��目录，然后运行以下命令，它会显示使用基本目录`/usr/local`启动服务器的效果：

    ```sql
    $> ./mysqld --basedir=/usr/local --verbose --help
    ```

    你可以指定其他选项，比如`--datadir`，但`--verbose`和`--help`必须是最后的选项。

    一旦确定了所需的路径设置，启动服务器时不要带`--verbose`和`--help`。

    如果**mysqld**当前正在运行，可以通过执行以下命令查找它正在使用的路径设置：

    ```sql
    $> mysqladmin variables
    ```

    或者：

    ```sql
    $> mysqladmin -h *host_name* variables
    ```

    *`host_name`*是 MySQL 服务器主机的名称。

+   确保服务器可以访问 data directory。数据目录及其内容的所有权和权限必须允许服务器读取和修改它们。

    如果在启动**mysqld**时遇到`Errcode 13`（表示`Permission denied`），这意味着数据目录或其内容的权限不允许服务器访问。在这种情况下，您需要更改相关文件和目录的权限，以便服务器有权使用它们。您也可以以`root`用户身份启动服务器，但这会带来安全问题，应该避免。

    切换到数据目录并检查数据目录及其内容的所有权，以确保服务器有访问权限。例如，如果数据目录是`/usr/local/mysql/var`，使用以下命令：

    ```sql
    $> ls -la /usr/local/mysql/var
    ```

    如果数据目录或其文件或子目录的所有权不属于您用于运行服务器的登录帐��，请将它们的所有权更改为该帐户。如果帐户名为`mysql`，请使用以下命令：

    ```sql
    $> chown -R mysql /usr/local/mysql/var
    $> chgrp -R mysql /usr/local/mysql/var
    ```

    即使拥有正确的所有权，如果系统上运行了其他安全软件来管理应用程序对文件系统各部分的访问，MySQL 可能无法启动。在这种情况下，重新配置该软件以允许**mysqld**访问其在正常操作期间使用的目录。

+   验证服务器要使用的网络接口是否可用。

    如果出现以下任一错误，则表示可能有其他程序（也许是另一个**mysqld**服务器）正在使用**mysqld**尝试使用的 TCP/IP 端口或 Unix 套接字文件：

    ```sql
    Can't start server: Bind on TCP/IP port: Address already in use
    Can't start server: Bind on unix socket...
    ```

    使用**ps**来确定是否有另一个**mysqld**服务器正在运行。如果是，请在再次启动**mysqld**之前关闭服务器。（如果有其他服务器在运行，并且您确实想要运行多个服务器，您可以在第 7.8 节，“在一台机器上运行多个 MySQL 实例”中找到如何操作的信息。）

    如果没有其他服务器在运行，请执行命令`telnet *`your_host_name`* *`tcp_ip_port_number`*`。（默认的 MySQL 端口号是 3306。）然后按下几次回车键。如果你没有收到类似`telnet: Unable to connect to remote host: Connection refused`的错误消息，那么可能是其他程序正在使用**mysqld**尝试使用的 TCP/IP 端口。找出是哪个程序在使用，并禁用它，或者告诉**mysqld**使用不同的端口监听，使用`--port`选项。在这种情况下，当使用 TCP/IP 连接到服务器时，客户端程序指定相同的非默认端口号。

    另一个端口无法访问的原因可能是您的防火墙阻止对其的连接。如果是这样，请修改防火墙设置以允许访问该端口。

    如果服务器启动但无法连接，请确保您在`/etc/hosts`中有类似以下内容的条目：

    ```sql
    127.0.0.1       localhost
    ```

+   如果无法启动**mysqld**，尝试使用`--debug`选项创建一个跟踪文件以查找问题。参见第 7.9.4 节，“DBUG 包”。
