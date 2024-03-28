# 8.7.2 更改 SELinux 模式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/selinux-mode.html`](https://dev.mysql.com/doc/refman/8.0/en/selinux-mode.html)

SELinux 支持强制执行、宽容和禁用模式。强制执行模式是默认模式。宽容模式允许在强制执行模式下不允许的操作，并将这些操作记录到 SELinux 审计日志中。宽容模式通常用于制定策略或故障排除。在禁用模式下，策略不会被执行，系统对象不会应用上下文，这使得以后启用 SELinux 变得困难。

要查看当前的 SELinux 模式，请使用之前提到的 **sestatus** 命令或 **getenforce** 实用程序。

```sql
$> getenforce
Enforcing
```

要更改 SELinux 模式，请使用 `setenforce` 实用程序：

```sql
$> setenforce 0
$> getenforce
Permissive
```

```sql
$> setenforce 1
$> getenforce
Enforcing
```

使用 **setenforce** 进行的更改在重新启动系统时会丢失。要永久更改 SELinux 模式，请编辑 `/etc/selinux/config` 文件并重新启动系统。
