> 原文：[`dev.mysql.com/doc/refman/8.0/en/too-many-connections.html`](https://dev.mysql.com/doc/refman/8.0/en/too-many-connections.html)

#### B.3.2.5 连接过多

如果客户端在尝试连接到**mysqld**服务器时遇到`连接过多`错误，那么所有可用连接都被其他客户端占用。

允许的连接数由`max_connections`系统变量控制。要支持更多连接，请将`max_connections`设置为更大的值。

**mysqld**实际上允许`max_connections` + 1 个客户端连接。额外的连接是为具有`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）的帐户保留的。通过授予管理员权限而不是普通用户（不应该需要该权限）的权限，管理员可以连接到服务器并使用`SHOW PROCESSLIST`来诊断问题，即使最大数量的非特权客户端已连接。请参阅 Section 15.7.7.29, “SHOW PROCESSLIST Statement”。

服务器还允许在专用接口上进行管理连接。有关服务器如何处理客户端连接的更多信息，请参阅 Section 7.1.12.1, “Connection Interfaces”。
