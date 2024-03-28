> 译文：[`dev.mysql.com/doc/refman/8.0/en/windows-testing.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-testing.html)

#### 2.3.4.9 测试 MySQL 安装

您可以通过执行以下任何命令来测试 MySQL 服务器是否正常工作：

```sql
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqlshow"
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqlshow" -u root mysql
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqladmin" version status proc
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysql" test
```

如果**mysqld**对来自客户端程序的 TCP/IP 连接响应缓慢，则您的 DNS 可能存在问题。在这种情况下，启动**mysqld**时启用`skip_name_resolve`系统变量，并且在 MySQL 授权表的`Host`列中仅使用`localhost`和 IP 地址。（确保存在指定 IP 地址的账户，否则可能无法连接。）

您可以通过指定`--pipe`或`--protocol=PIPE`选项，或通过将`.`（句点）作为主机名来强制 MySQL 客户端使用命名管道连接而不是 TCP/IP。如果您不想使用默认管道名称，请使用`--socket`选项指定管道的名称。

如果您已为`root`账户设置了密码，删除了匿名账户，或创建了新用户账户，则连接到 MySQL 服务器时必须使用先前显示的命令和适当的`-u`和`-p`选项。请参阅第 6.2.4 节，“使用命令选项连接到 MySQL 服务器”。

有关**mysqlshow**的更多信息，请参阅第 6.5.7 节，“mysqlshow — 显示数据库、表和列信息”。
