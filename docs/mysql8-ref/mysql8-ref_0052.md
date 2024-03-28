> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-start-command-line.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-start-command-line.html)

#### 2.3.4.6 从 Windows 命令行启动 MySQL

MySQL 服务器可以从命令行手动启动。这可以在任何版本的 Windows 上完成。

要从命令行启动**mysqld**服务器，您应该启动控制台窗口（或“DOS 窗口”）并输入此命令：

```sql
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqld"
```

**mysqld**的路径可能会根据系统上 MySQL 的安装位置而有所不同。

您可以通过执行此命令来停止 MySQL 服务器：

```sql
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqladmin" -u root shutdown
```

注意

如果 MySQL `root`用户帐户有密码，则需要使用`-p`选项调用**mysqladmin**并在提示时提供密码。

此命令调用 MySQL 管理实用程序**mysqladmin**连接到服务器并告诉其关闭。该命令以 MySQL `root`用户身份连接，这是 MySQL 授权系统中的默认管理帐户。

注意

MySQL 授权系统中的用户与 Microsoft Windows 下的任何操作系统用户完全独立。

如果**mysqld**无法启动，请检查错误日志，看看服务器是否在那里写入任何消息以指示问题的原因。默认情况下，错误日志位于`C:\Program Files\MySQL\MySQL Server 8.0\data`目录中。它是带有`.err`后缀的文件，或者可以通过传递`--log-error`选项来指定。或者，您可以尝试使用`--console`选项启动服务器；在这种情况下，服务器可能会在屏幕上显示一些有用的信息以帮助解决问题。

最后一个选项是使用`--standalone`和`--debug`选项启动**mysqld**。在这种情况下，**mysqld**会写入一个日志文件`C:\mysqld.trace`，其中应包含**mysqld**无法启动的原因。参见第 7.9.4 节，“DBUG 包”。

使用**mysqld --verbose --help**显示**mysqld**支持的所有选项。
