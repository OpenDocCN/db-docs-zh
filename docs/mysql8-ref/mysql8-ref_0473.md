# 8.7 SELinux

> 原文：[`dev.mysql.com/doc/refman/8.0/en/selinux.html`](https://dev.mysql.com/doc/refman/8.0/en/selinux.html)

8.7.1 检查 SELinux 是否已启用

8.7.2 更改 SELinux 模式

8.7.3 MySQL 服务器 SELinux 策略

8.7.4 SELinux 文件上下文

8.7.5 SELinux TCP 端口上下文

8.7.6 SELinux 故障排除

安全增强型 Linux（SELinux）是一个强制访问控制（MAC）系统，通过为每个系统对象应用一个称为*SELinux 上下文*的安全标签来实现访问权限。SELinux 策略模块使用 SELinux 上下文来定义进程、文件、端口和其他系统对象之间如何相互交互的规则。只有在策略规则允许的情况下，系统对象之间的交互才被允许。

SELinux 上下文（应用于系统对象的标签）具有以下字段：`user`、`role`、`type` 和 `security level`。最常用于定义进程如何与其他系统对象交互的规则的是类型信息，而不是整个 SELinux 上下文。例如，MySQL SELinux 策略模块使用 `type` 信息定义策略规则。

您可以使用操作系统命令（如 **ls** 和 **ps**）以 `-Z` 选项查看 SELinux 上下文。假设 SELinux 已启用并且 MySQL 服务器正在运行，则以下命令显示 **mysqld** 进程和 MySQL 数据目录的 SELinux 上下文：

**mysqld** 进程：

```sql
$> ps -eZ | grep mysqld
system_u:system_r:mysqld_t:s0    5924 ?        00:00:03 mysqld
```

MySQL 数据目录：

```sql
$> cd /var/lib
$> ls -Z | grep mysql
system_u:object_r:mysqld_db_t:s0 mysql
```

其中：

+   `system_u` 是用于系统进程和对象的 SELinux 用户标识。

+   `system_r` 是用于系统进程的 SELinux 角色。

+   `objects_r` 是用于系统对象的 SELinux 角色。

+   `mysqld_t` 是与 mysqld 进程关联的类型。

+   `mysqld_db_t` 是与 MySQL 数据目录及其文件关联的类型。

+   `s0` 是安全级别。

有关解释 SELinux 上下文的更多信息，请参考您发行版的 SELinux 文档。
