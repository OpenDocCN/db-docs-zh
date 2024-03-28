# 8.7.5 SELinux TCP 端口上下文

> 原文：[`dev.mysql.com/doc/refman/8.0/en/selinux-context-tcp-port.html`](https://dev.mysql.com/doc/refman/8.0/en/selinux-context-tcp-port.html)

8.7.5.1 设置 mysqld 的 TCP 端口上下文

8.7.5.2 设置 MySQL 功能端口的 TCP 端口上下文

接下来的说明将使用`semanage`二进制文件来管理端口上下文；在 RHEL 上，它是`policycoreutils-python-utils`软件包的一部分。

```sql
yum install -y policycoreutils-python-utils
```

安装完`semanage`二进制文件后，您可以使用`semanage`的`port`选项列出使用`mysqld_port_t`上下文定义的端口。

```sql
$> semanage port -l | grep mysqld
mysqld_port_t                  tcp      1186, 3306, 63132-63164
```
