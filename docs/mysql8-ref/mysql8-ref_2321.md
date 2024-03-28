> 原文：[`dev.mysql.com/doc/refman/8.0/en/problems-with-mysql-sock.html`](https://dev.mysql.com/doc/refman/8.0/en/problems-with-mysql-sock.html)

#### B.3.3.6 如何保护或更改 MySQL Unix 套接字文件

服务器用于与本地客户端通信的 Unix 套接字文件的默认位置是`/tmp/mysql.sock`。（对于某些分发格式，目录可能不同，例如 RPM 的`/var/lib/mysql`。）

在某些 Unix 版本上，任何人都可以删除`/tmp`目录或其他用于临时文件的类似目录中的文件。如果套接字文件位于系统上的此类目录中，可能会导致问题。

在大多数 Unix 版本上，您可以保护`/tmp`目录，以便文件只能被其所有者或超级用户（`root`）删除。要做到这一点，请以`root`身份登录并使用以下命令在`/tmp`目录上设置`sticky`位：

```sql
$> chmod +t /tmp
```

您可以通过执行`ls -ld /tmp`来检查`sticky`位是否已设置。如果最后一个权限字符是`t`，则该位已设置。

另一种方法是更改服务器创建 Unix 套接字文件的位置。如果这样做，您还应该让客户端程序知道文件的新位置。您可以通过几种方式指定文件位置：

+   在全局或本地选项文件中指定路径。例如，在`/etc/my.cnf`中放入以下行：

    ```sql
    [mysqld]
    socket=/path/to/socket

    [client]
    socket=/path/to/socket
    ```

    参见第 6.2.2.2 节，“使用选项文件”。

+   在命令行上指定`--socket`选项给**mysqld_safe**以及运行客户端程序时。

+   将`MYSQL_UNIX_PORT`环境变量设置为 Unix 套接字文件的路径。

+   重新编译 MySQL 源代码以使用不同的默认 Unix 套接字文件位置。在运行**CMake**时，使用`MYSQL_UNIX_ADDR`选项定义文件的路径。参见第 2.8.7 节，“MySQL 源配置选项”。

您可以通过尝试使用此命令连接到服务器来测试新的套接字位置是否有效：

```sql
$> mysqladmin --socket=/path/to/socket version
```
