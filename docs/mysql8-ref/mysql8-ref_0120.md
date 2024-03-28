# 5.1 连接和断开服务器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/connecting-disconnecting.html`](https://dev.mysql.com/doc/refman/8.0/en/connecting-disconnecting.html)

要连接到服务器，通常需要在调用**mysql**时提供一个 MySQL 用户名，并且很可能需要一个密码。如果服务器运行在您登录的机器之外的机器上，您还必须指定主机名。联系管理员以找出应该使用哪些连接参数来连接（即使用哪个主机、用户名和密码）。一旦知道正确的参数，您应该能够像这样连接：

```sql
$> mysql -h *host* -u *user* -p
Enter password: ********
```

*`host`* 和 *`user`* 分别代表您的 MySQL 服务器运行的主机名和您的 MySQL 帐户的用户名。替换适合您设置的值。`********`代表您的密码；当**mysql**显示`输入密码：`提示时输入。

如果一切正常，您应该看到一些简介信息，然后是一个`mysql>`提示符：

```sql
$> mysql -h *host* -u *user* -p
Enter password: ********
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 25338 to server version: 8.0.36-standard

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql>
```

`mysql>`提示符告诉您**mysql**已准备好让您输入 SQL 语句。

如果您正在登录与 MySQL 运行在同一台机器上，您可以省略主机，并简单地使用以下内容：

```sql
$> mysql -u *user* -p
```

如果在尝试登录时出现错误消息，例如 ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)，这意味着 MySQL 服务器守护程序（Unix）或服务（Windows）未运行。请咨询管理员或查看适用于您操作系统的第二章 *安装 MySQL*部分。

如果在尝试登录时遇到其他常见问题，可以参考第 B.3.2 节 “使用 MySQL 程序时的常见错误”进行帮助。

一些 MySQL 安装允许用户以匿名（未命名）用户连接到在本地主机上运行的服务器。如果您的机器是这种情况，您应该能够通过调用**mysql**而不使用任何选项来连接到该服务器：

```sql
$> mysql
```

成功连接后，您可以随时在`mysql>`提示符处键入`QUIT`（或`\q`）来断开连接：

```sql
mysql> QUIT
Bye
```

在 Unix 上，您也可以通过按下 Control+D 来断开连接。

在接下来的章节中，大多数示例假定您已连接到服务器。它们通过`mysql>`提示符来指示这一点。
