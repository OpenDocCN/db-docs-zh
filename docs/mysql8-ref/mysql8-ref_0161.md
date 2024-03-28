# 6.2.4 使用命令选项连接到 MySQL 服务器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/connecting.html`](https://dev.mysql.com/doc/refman/8.0/en/connecting.html)

本节描述了使用命令行选项来指定如何建立与 MySQL 服务器的连接，适用于诸如**mysql**或**mysqldump**等客户端。有关使用类似 URI 的连接字符串或键值对建立连接的信息，适用于 MySQL Shell 等客户端，请参阅第 6.2.5 节，“使用类似 URI 的字符串或键值对连接到服务器”。如果无法连接，需要更多信息，请参阅第 8.2.22 节，“解决连接到 MySQL 的问题”。

要使客户端程序连接到 MySQL 服务器，必须使用正确的连接参数，如运行服务器的主机名以及您的 MySQL 帐户的用户名和密码。每个连接参数都有一个默认值，但您可以使用程序选项根据需要覆盖默认值，这些选项可以在命令行上指定或在选项文件中指定。

这里的示例使用**mysql**客户端程序，但原则适用于其他客户端，如**mysqldump**、**mysqladmin**或**mysqlshow**。

此命令调用**mysql**而不指定任何显式连接参数：

```sql
mysql
```

因为没有参数选项，所以适用默认值：

+   默认主机名是`localhost`。在 Unix 上，这具有特殊含义，稍后会描述。

+   默认用户名是 Windows 上的`ODBC`或 Unix 上的登录名。

+   没有发送密码，因为既没有`--password`也没有给出`-p`。

+   对于**mysql**，第一个非选项参数被视为默认数据库的名称。因为没有这样的参数，**mysql**选择不使用默认数据库。

要明确指定主机名和用户名，以及密码，请在命令行上提供适当的选项。要选择默认数据库，请添加一个数据库名称参数。示例：

```sql
mysql --host=localhost --user=myname --password=*password* mydb
mysql -h localhost -u myname -p*password* mydb
```

对于密码选项，密码值是可选的：

+   如果使用`--password`或`-p`选项并指定密码值，则`--password=`或`-p`与其后的密码之间*不能有空格*。

+   如果你使用`--password`或`-p`但没有指定密码值，客户端程序会提示你输入密码。在输入密码时，密码不会显示出来。这比在命令行上提供密码更安全，因为这样可能会使系统上的其他用户通过执行诸如**ps**之类的命令看到密码。参见第 8.1.2.1 节，“终端用户密码安全指南”。

+   要明确指定没有密码，并且客户端程序不应提示输入密码，请使用`--skip-password`选项。

正如刚才提到的，将密码值包含在命令行上是一种安全风险。为了避免这种风险，请指定不带任何后续密码值的`--password`或`-p`选项：

```sql
mysql --host=localhost --user=myname --password mydb
mysql -h localhost -u myname -p mydb
```

当使用`--password`或`-p`选项时，没有密码值，客户端程序会打印提示并等待你输入密码。（在这些示例中，`mydb`并不被解释为密码，因为它与前面的密码选项之间有一个空格。）

在某些系统上，MySQL 用于提示密码的库例程会自动将密码限制为八个字符。这个限制是系统库的属性，而不是 MySQL 的属性。在内部，MySQL 对密码长度没有任何限制。为了解决受影响系统上的限制，可以在选项文件中指定密码（参见第 6.2.2.2 节，“使用选项文件”）。另一个解决方法是将 MySQL 密码更改为八个或更少字符的值，但这样做的缺点是较短的密码往往不够安全。

客户端程序确定要进行的连接类型如下：

+   如果未指定主机或主机为`localhost`，则会连接到本地主机：

    +   在 Windows 上，如果服务器启动时启用了支持共享内存连接的`shared_memory`系统变量，则客户端将使用共享内存连接。

    +   在 Unix 上，MySQL 程序对主机名`localhost`进行了特殊处理，这与您对其他基于网络的程序的期望可能不同：客户端使用 Unix 套接字文件进行连接。可以使用`--socket`选项或`MYSQL_UNIX_PORT`环境变量来指定套接字名称。

+   在 Windows 上，如果`host`是`.`（句点），或者未启用 TCP/IP 并且未指定`--socket`或主机名为空，则客户端将使用命名管道连接，如果服务器启用了`named_pipe`系统变量以支持命名管道连接。如果不支持命名管道连接，或者进行连接��用户不是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员，则会发生错误。

+   否则，连接将使用 TCP/IP。

`--protocol`选项使您能够即使其他选项通常会导致使用不同协议，也可以使用特定传输协议。也就是说，`--protocol`明确指定传输协议并覆盖前述规则，即使是对于`localhost`。

只有与所选传输协议相关的连接选项才会被使用或检查。其他连接选项将被忽略。例如，在 Unix 上使用`--host=localhost`，客户端尝试使用 Unix 套接字文件连接到本地服务器，即使给定了`--port`或`-P`选项来指定 TCP/IP 端口号。

为确保客户端与本地服务器建立 TCP/IP 连接，请使用`--host`或`-h`来指定主机名值为`127.0.0.1`（而不是`localhost`），或本地服务器的 IP 地址或名称。您还可以明确指定传输协议，即使是对于`localhost`，也可以使用`--protocol=TCP`选项。示例：

```sql
mysql --host=127.0.0.1
mysql --protocol=TCP
```

如果服务器配置为接受 IPv6 连接，则客户端可以使用`--host=::1`通过 IPv6 连接到本地服务器。请参阅第 7.1.13 节，“IPv6 支持”。

在 Windows 上，要强制 MySQL 客户端使用命名管道连接，请指定`--pipe`或`--protocol=PIPE`选项，或将`.`（句点）作为主机名。如果服务器未启用`named_pipe`系统变量以支持命名管道连接，或者进行连接的用户不是由`named_pipe_full_access_group`系统变量指定的 Windows 组的成员，则会发生错误。如果不想使用默认管道名称，请使用`--socket`选项指定管道的名称。

连接到远程服务器使用 TCP/IP。此命令连接到运行在`remote.example.com`上的服务器，使用默认端口号（3306）：

```sql
mysql --host=remote.example.com
```

要明确指定端口号，请使用`--port`或`-P`选项：

```sql
mysql --host=remote.example.com --port=13306
```

您也可以为连接到本地服务器指定端口号。但是，如前所述，在 Unix 上连接到`localhost`默认使用套接字文件，因此除非您像之前描述的那样强制使用 TCP/IP 连接，否则任何指定端口号的选项都将被忽略。

对于此命令，在 Unix 上程序使用套接字文件，`--port`选项将被忽略：

```sql
mysql --port=13306 --host=localhost
```

要使用端口号，请强制使用 TCP/IP 连接。例如，以以下任一种方式调用程序：

```sql
mysql --port=13306 --host=127.0.0.1
mysql --port=13306 --protocol=TCP
```

有关控制客户端程序如何与服务器建立连接的选项的更多信息，请参见第 6.2.3 节，“连接到服务器的命令选项”。

可以在每次调用客户端程序时指定连接参数，而无需每次在命令行中输入它们：

+   在选项文件的`[client]`部分指定连接参数。文件的相关部分可能如下所示：

    ```sql
    [client]
    host=*host_name*
    user=*user_name*
    password=*password*
    ```

    查看更多信息，请参见第 6.2.2.2 节，“使用选项文件”。

+   一些连接参数可以使用环境变量指定。例如：

    +   要指定**mysql**的主机，请使用`MYSQL_HOST`。

    +   在 Windows 上，要指定 MySQL 用户名，请使用`USER`。

    要查看支持的环境变量列表，请参见第 6.9 节，“环境变量”。
