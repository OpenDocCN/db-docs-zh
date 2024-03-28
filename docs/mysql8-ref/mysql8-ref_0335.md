# 7.8.4 在多服务器环境中使用客户端程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/multiple-server-clients.html`](https://dev.mysql.com/doc/refman/8.0/en/multiple-server-clients.html)

要连接到一个 MySQL 服务器的客户端程序，该服务器监听的网络接口与编译到您的客户端的不同，您可以使用以下方法之一：

+   使用`--host=*`host_name`*` `--port=*`port_number`*`启动客户端，使用 TCP/IP 连接到远程服务器，使用`--host=127.0.0.1` `--port=*`port_number`*`使用 TCP/IP 连接到本地服务器，或使用`--host=localhost` `--socket=*`file_name`*`连接到本地服务器使用 Unix 套接字文件或 Windows 命名管道。

+   启动客户端时使用`--protocol=TCP`以使用 TCP/IP 进行连接，使用`--protocol=SOCKET`以使用 Unix 套接字文件进行连接，使用`--protocol=PIPE`以使用命名管道进行连接，或使用`--protocol=MEMORY`以使用共享内存进行连接。对于 TCP/IP 连接，您可能还需要指定`--host`和`--port`选项。对于其他类型的连接，您可能需要指定`--socket`选项以指定 Unix 套接字文件或 Windows 命名管道名称，或者指定`--shared-memory-base-name`选项以指定共享内存名称。共享内存连接仅在 Windows 上受支持。

+   在 Unix 上，在启动客户端之前，将`MYSQL_UNIX_PORT`和`MYSQL_TCP_PORT`环境变量设置为指向 Unix 套接字文件和 TCP/IP 端口号。如果您通常使用特定的套接字文件或端口号，您可以将设置这些环境变量的命令放在您的`.login`文件中，以便每次登录时应用。参见第 6.9 节，“环境变量”。

+   在选项文件的`[client]`组中指定默认的 Unix 套接字文件和 TCP/IP 端口号。例如，您可以在 Windows 上使用`C:\my.cnf`，或在 Unix 上使用您的主目录中的`.my.cnf`文件。参见第 6.2.2.2 节，“使用选项文件”。

+   在 C 程序中，你可以在`mysql_real_connect()`调用中指定套接字文件或端口号参数。你也可以通过调用`mysql_options()`让程序读取选项文件。参见 C API 基本函数描述。

+   如果你正在使用 Perl 的`DBD::mysql`模块，你可以从 MySQL 选项文件中读取选项。例如：

    ```sql
    $dsn = "DBI:mysql:test;mysql_read_default_group=client;"
            . "mysql_read_default_file=/usr/local/mysql/data/my.cnf";
    $dbh = DBI->connect($dsn, $user, $password);
    ```

    查看第 31.9 节，“MySQL Perl API”。

    其他编程接口可能提供类似的功能来读取选项文件。
