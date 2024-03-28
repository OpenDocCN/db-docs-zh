# 8.3.4 通过 SSH 从 Windows 远程连接到 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-and-ssh.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-and-ssh.html)

本节描述了如何通过 SSH 与远程 MySQL 服务器建立加密连接。这些信息由 David Carlson 提供 `<dcarlson@mplcomm.com>`。

1.  在您的 Windows 机器上安装一个 SSH 客户端。有关 SSH 客户端的比较，请参见[`en.wikipedia.org/wiki/Comparison_of_SSH_clients`](http://en.wikipedia.org/wiki/Comparison_of_SSH_clients)。

1.  启动您的 Windows SSH 客户端。将`Host_Name = *`yourmysqlserver_URL_or_IP`*`设置为您的 MySQL 服务器的 URL 或 IP。将`userid=*`your_userid`*`设置为登录到服务器的用户 ID。这个`userid`值可能与您的 MySQL 账户的用户名不同。

1.  设置端口转发。可以进行远程转发（设置`local_port: 3306`，`remote_host: *`yourmysqlservername_or_ip`*`，`remote_port: 3306`）或本地转发（设置`port: 3306`，`host: localhost`，`remote port: 3306`）。

1.  保存所有设置，否则下次必须重新设置。

1.  使用您刚刚创建的 SSH 会话登录到服务器。

1.  在您的 Windows 机器上，启动一些 ODBC 应用程序（如 Access）。

1.  在 Windows 中创建一个新文件，并以与通常相同的方式链接到 MySQL，只是在 MySQL 主机服务器处键入`localhost`，而不是*`yourmysqlservername`*。

此时，您应该已经建立了使用 SSH 加密的 MySQL 的 ODBC 连接。
