# 28.6.2 INFORMATION_SCHEMA CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-connection-control-failed-login-attempts-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-connection-control-failed-login-attempts-table.html)

该表提供有关每个帐户（用户/主机组合）的当前连续失败连接尝试次数的信息。

`CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS`具有以下列：

+   `USERHOST`

    表示已经失败连接尝试的帐户的用户/主机组合，格式为`'*`user_name`*'@'*`host_name`*'`。

+   `FAILED_ATTEMPTS`

    `USERHOST`值的当前连续失败连接尝试次数。这计算所有失败尝试，无论是否延迟。服务器为其响应添加延迟的尝试次数是`FAILED_ATTEMPTS`值与`connection_control_failed_connections_threshold`系统变量值之间的差异。

#### 注：

+   必须激活`CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS`插件才能使用此表，并且必须激活`CONNECTION_CONTROL`插件，否则表内容始终为空。请参阅 Section 8.4.2, “The Connection-Control Plugins”。

+   该表仅包含对于已经有一个或多个连续失败连接尝试而没有随后成功尝试的帐户的行。当帐户成功连接时，其失败连接计数将重置为零，并且服务器将删除与该帐户对应的任何行。

+   在运行时为`connection_control_failed_connections_threshold`系统变量分配一个值会将所有累积的失败连接计数器重置为零，导致表变为空。
