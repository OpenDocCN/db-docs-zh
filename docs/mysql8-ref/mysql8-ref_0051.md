> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-server-first-start.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-server-first-start.html)

#### 2.3.4.5 首次启动服务器

本节概述了启动 MySQL 服务器的一般概况。以下各节提供了更具体的信息，用于从命令行启动 MySQL 服务器或作为 Windows 服务。

这里的信息主要适用于使用`noinstall`版本安装 MySQL，或者如果您希望手动配置和测试 MySQL 而不是使用 MySQL 安装程序。

这些部分中的示例假定 MySQL 安装在默认位置`C:\Program Files\MySQL\MySQL Server 8.0`下。如果您将 MySQL 安装在其他位置，请调整示例中显示的路径名。

客户端有两个选项。他们可以使用 TCP/IP，或者如果服务器支持命名管道连接，他们可以使用命名管道。

对于 Windows 的 MySQL 还支持共享内存连接，如果服务器启用了`shared_memory`系统变量。客户端可以通过使用`--protocol=MEMORY`选项通过共享内存连接。

有关要运行哪个服务器二进制文件的信息，请参阅 Section 2.3.4.3, “Selecting a MySQL Server Type”。

最好从控制台窗口的命令提示符中进行测试（或“DOS 窗口”）。这样，您可以在易于查看的窗口中让服务器显示状态消息。如果您的配置有问题，这些消息将使您更容易识别和解决任何问题。

注意

必须在启动 MySQL 之前初始化数据库。有关初始化过程的其他信息，请参阅 Section 2.9.1, “Initializing the Data Directory”。

要启动服务器，请输入以下命令：

```sql
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqld" --console
```

当服务器启动时，您应该看到类似以下的消息（路径名和大小可能不同）。`ready for connections`消息表示服务器已准备好为客户端连接提供服务。

```sql
[Server] C:\mysql\bin\mysqld.exe (mysqld 8.0.30) starting as process 21236
[InnoDB] InnoDB initialization has started.
[InnoDB] InnoDB initialization has ended.
[Server] CA certificate ca.pem is self signed.
[Server] Channel mysql_main configured to support TLS. 
Encrypted connections are now supported for this channel.
[Server] X Plugin ready for connections. Bind-address: '::' port: 33060
[Server] C:\mysql\bin\mysqld.exe: ready for connections. 
Version: '8.0.30'  socket: ''  port: 3306  MySQL Community Server - GPL.
```

您现在可以打开一个新的控制台窗口来运行客户端程序。

如果省略`--console`选项，服务器将将诊断输出写入数据目录中的错误日志（默认为`C:\Program Files\MySQL\MySQL Server 8.0\data`）。错误日志是扩展名为`.err`的文件，并且可以使用`--log-error`选项进行设置。

注意

MySQL 授权表中的初始`root`账户没有密码。启动服务器后，您应该按照 Section 2.9.4, “Securing the Initial MySQL Account”中的说明为其设置密码。
