# 8.7.3 MySQL 服务器 SELinux 策略

> 原文：[`dev.mysql.com/doc/refman/8.0/en/selinux-policies.html`](https://dev.mysql.com/doc/refman/8.0/en/selinux-policies.html)

MySQL 服务器 SELinux 策略模块通常默认安装。您可以使用**semodule -l**命令查看已安装的模块。MySQL 服务器 SELinux 策略模块包括：

+   `mysqld_selinux`

+   `mysqld_safe_selinux`

有关 MySQL 服务器 SELinux 策略模块的信息，请参考 SELinux 手册页面。手册页面提供有关与 MySQL 服务相关的类型和布尔值的信息。手册页面的命名格式为`*`service-name`*_selinux`。

```sql
man mysqld_selinux
```

如果没有 SELinux 手册页面，请参考您的发行版 SELinux 文档，了解如何使用`sepolicy manpage`工具生成手册页面的信息。
