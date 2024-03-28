# 7.8.3 在 Unix 上运行多个 MySQL 实例

> 原文：[`dev.mysql.com/doc/refman/8.0/en/multiple-unix-servers.html`](https://dev.mysql.com/doc/refman/8.0/en/multiple-unix-servers.html)

注意

这里的讨论使用 **mysqld_safe** 来启动多个 MySQL 实例。对于使用 RPM 发行版的 MySQL 安装，在几个 Linux 平台上，服务器的启动和关闭由 systemd 管理。在这些平台上，**mysqld_safe** 没有安装，因为它是不必要的。有关使用 systemd 处理多个 MySQL 实例的信息，请参见 Section 2.5.9, “Managing MySQL Server with systemd”。

在 Unix 上运行多个 MySQL 实例的一种方法是使用不同的默认 TCP/IP 端口和 Unix 套接字文件编译不同的服务器，以便每个服务器在不同的网络接口上监听。为每个安装编译不同的基目录还会自动为每个服务器生成单独的编译数据目录、日志文件和 PID 文件位置。

假设现有的 5.7 服务器配置为默认的 TCP/IP 端口号（3306）和 Unix 套接字文件（`/tmp/mysql.sock`）。要配置一个新的 8.0.36 服务器具有不同的操作参数，可以使用类似以下的 **CMake** 命令：

```sql
$> cmake . -DMYSQL_TCP_PORT=*port_number* \
             -DMYSQL_UNIX_ADDR=*file_name* \
             -DCMAKE_INSTALL_PREFIX=/usr/local/mysql-8.0.36
```

这里，*`port_number`* 和 *`file_name`* 必须与默认的 TCP/IP 端口号和 Unix 套接字文件路径名不同，并且 `CMAKE_INSTALL_PREFIX` 的值应指定一个安装目录，该目录与现有 MySQL 安装所在的目录不同。

如果您有一个 MySQL 服务器在特定端口号上监听，您可以使用以下命令查找它用于几个重要可配置变量的操作参数，包括基目录和 Unix 套接字文件名：

```sql
$> mysqladmin --host=*host_name* --port=*port_number* variables
```

通过该命令显示的信息，您可以知道在配置额外服务器时不应使用哪些选项值。

如果将 `localhost` 指定为主机名，**mysqladmin** 默认使用 Unix 套接字文件而不是 TCP/IP。要明确指定传输协议，请使用 `--protocol={TCP|SOCKET|PIPE|MEMORY}` 选项。

您无需编译一个新的 MySQL 服务器只是为了使用不同的 Unix 套接字文件和 TCP/IP 端口号开始。也可以使用相同的服务器二进制文件，并在运行时使用不同的参数值启动每个调用。一种方法是使用命令行选项：

```sql
$> mysqld_safe --socket=*file_name* --port=*port_number*
```

要启动第二个服务器，请提供不同的`--socket` 和 `--port` 选项值，并通过`--datadir=*`dir_name`*` 选项传递给 **mysqld_safe**，以便服务器使用不同的数据目录。

或者，将每个服务器的选项放在不同的选项文件中，然后使用`--defaults-file` 选项启动每个服务器，该选项指定适当选项文件的路径。例如，如果两个服务器实例的选项文件分别命名为 `/usr/local/mysql/my.cnf` 和 `/usr/local/mysql/my.cnf2`，则可以这样启动服务器：命令：

```sql
$> mysqld_safe --defaults-file=/usr/local/mysql/my.cnf
$> mysqld_safe --defaults-file=/usr/local/mysql/my.cnf2
```

实现类似效果的另一种方法是使用环境变量设置 Unix 套接字文件名和 TCP/IP 端口号：

```sql
$> MYSQL_UNIX_PORT=/tmp/mysqld-new.sock
$> MYSQL_TCP_PORT=3307
$> export MYSQL_UNIX_PORT MYSQL_TCP_PORT
$> bin/mysqld --initialize --user=mysql
$> mysqld_safe --datadir=/path/to/datadir &
```

这是一个快速启动第二个用于测试的服务器的方法。这种方法的好处是环境变量设置适用于从同一 shell 中调用的任何客户端程序。因此，这些客户端的连接会自动定向到第二个服务器。

Section 6.9, “环境变量” 包括了您可以使用来影响 MySQL 程序的其他环境变量列表。

在 Unix 系统上，**mysqld_multi** 脚本提供了另一种启动多个服务器的方式。请参阅 Section 6.3.4, “mysqld_multi — 管理多个 MySQL 服务器”。
