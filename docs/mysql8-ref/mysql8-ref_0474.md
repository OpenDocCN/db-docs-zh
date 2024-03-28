# 8.7.1 检查 SELinux 是否已启用

> 原文：[`dev.mysql.com/doc/refman/8.0/en/selinux-checking.html`](https://dev.mysql.com/doc/refman/8.0/en/selinux-checking.html)

在一些 Linux 发行版上，默认启用 SELinux，包括 Oracle Linux、RHEL、CentOS 和 Fedora。使用 **sestatus** 命令来确定您的发行版是否已启用 SELinux：

```sql
$> sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      31
```

如果 SELinux 已禁用或找不到 **sestatus** 命令，请在启用 SELinux 之前参考您的发行版 SELinux 文档以获取指导。
