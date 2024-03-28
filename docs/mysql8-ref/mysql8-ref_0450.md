> 原文：[`dev.mysql.com/doc/refman/8.0/en/firewall-elements.html`](https://dev.mysql.com/doc/refman/8.0/en/firewall-elements.html)

#### 8.4.7.1 MySQL 企业防火墙的元素

MySQL 企业防火墙基于一个包含以下元素的插件库：

+   一个名为`MYSQL_FIREWALL`的服务器端插件在 SQL 语句执行之前检查，并根据注册的防火墙配置文件，决定是否执行或拒绝每个语句。

+   `MYSQL_FIREWALL`插件，以及名为`MYSQL_FIREWALL_USERS`和`MYSQL_FIREWALL_WHITELIST`的服务器端插件实现了性能模式和`INFORMATION_SCHEMA`表，提供了对注册配置文件的视图。

+   配置文件在内存中缓存以提高性能。`mysql`系统数据库中的表提供了防火墙数据的后备存储，以确保配置文件在服务器重新启动时持久化。

+   存储过程执行诸如注册防火墙配置文件、建立其操作模式以及管理缓存与持久存储之间的防火墙数据传输等任务。

+   管理功能提供了一个 API 用于较低级别的任务，比如将缓存与持久存储同步。

+   系统变量使防火墙配置生效，状态变量提供运行时操作信息。

+   `FIREWALL_ADMIN`和`FIREWALL_USER`权限使用户能够管理任何用户的防火墙规则，以及他们自己的防火墙规则。

+   `FIREWALL_EXEMPT`权限（自 MySQL 8.0.27 起可用）免除用户的防火墙限制。例如，对于配置防火墙的任何数据库管理员来说，这是有用的，以避免错误配置导致管理员被锁定并无法执行语句的可能性。
