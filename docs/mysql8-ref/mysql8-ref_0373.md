# 8.2.13 权限更改何时生效

> 原文：[`dev.mysql.com/doc/refman/8.0/en/privilege-changes.html`](https://dev.mysql.com/doc/refman/8.0/en/privilege-changes.html)

如果**mysqld**服务器在没有`--skip-grant-tables`选项的情况下启动，它在启动序列期间将所有授权表内容读入内存。在那一点上，内存中的表对访问控制生效。

如果您间接修改授权表，使用帐户管理语句，服务器会立即注意到这些更改并重新加载授权表到内存中。帐户管理语句在第 15.7.1 节“帐户管理语句”中描述。例如包括`GRANT`、`REVOKE`、`SET PASSWORD`和`RENAME USER`。

如果您直接修改授权表，使用诸如`INSERT`、`UPDATE`或`DELETE`等语句（不建议这样做），则在告知服务器重新加载表或重新启动之前，更改不会影响权限检查。因此，如果您直接更改授权表但忘记重新加载它们，更改将*不会生效*，直到重新启动服务器。这可能会让您想知道为什么您的更改似乎没有任何影响！

要告知服务器重新加载授权表，请执行刷新权限操作。这可以通过发出`FLUSH PRIVILEGES`语句或执行**mysqladmin flush-privileges**或**mysqladmin reload**命令来完成。

重新加载授权表会影响每个现有客户端会话的权限：

+   表和列权限更改将在客户端的下一个请求中生效。

+   数据库权限更改将在客户端执行`USE *`db_name`*`语句的下一次生效。

    注意

    客户端应用程序可能会缓存数据库名称；因此，这种效果可能对它们不可见，除非实际更改为不同的数据库。

+   静态全局权限和密码对已连接的客户端不受影响。这些更改仅在后续连接的会话中生效。动态全局权限的更改立即生效。有关静态和动态权限之间的区别，请参阅静态与动态权限。

会话中活动角色集的更改立即生效，仅对该会话有效。`SET ROLE`语句执行会话角色的激活和停用（参见第 15.7.1.11 节，“SET ROLE Statement”）。

如果服务器使用`--skip-grant-tables`选项启动，则不会读取授权表或实施任何访问控制。任何用户都可以连接并执行任何操作，*这是不安全的*。要使这样启动的服务器读取表并启用访问检查，请刷新权限。
