# 8.4.2 连接控制插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/connection-control.html`](https://dev.mysql.com/doc/refman/8.0/en/connection-control.html)

8.4.2.1 连接控制插件安装

8.4.2.2 连接控制系统和状态变量

MySQL 服务器包含一个插件库，使管理员能够在一定数量的连续失败尝试之后，向连接尝试的服务器响应引入逐渐增加的延迟。这种能力提供了一个减缓措施，可以减缓针对 MySQL 用户账户的暴力攻击。插件库包含两个插件：

+   `CONNECTION_CONTROL` 检查传入的连接尝试，并根据需要向服务器响应添加延迟。该插件还公开了系统变量，使其操作可以配置，并提供了一个状态变量，提供基本的监控信息。

    `CONNECTION_CONTROL` 插件使用审计插件接口（参见 编写审计插件）。为了收集信息，它订阅了 `MYSQL_AUDIT_CONNECTION_CLASSMASK` 事件类，并处理 `MYSQL_AUDIT_CONNECTION_CONNECT` 和 `MYSQL_AUDIT_CONNECTION_CHANGE_USER` 子事件，以检查服务器是否应在响应连接尝试之前引入延迟。

+   `CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS` 实现了一个 `INFORMATION_SCHEMA` 表，公开了有关失败连接尝试的更详细的监控信息。

以下各节提供有关连接控制插件安装和配置的信息。有关 `CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS` 表的信息，请参阅 第 28.6.2 节，“INFORMATION_SCHEMA CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS 表”。
