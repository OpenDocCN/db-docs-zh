> 原文：[`dev.mysql.com/doc/refman/8.0/en/selinux-context-mysqld-tcp-port.html`](https://dev.mysql.com/doc/refman/8.0/en/selinux-context-mysqld-tcp-port.html)

#### 8.7.5.1 为 mysqld 设置 TCP 端口上下文

默认的 TCP 端口**mysqld**是`3306`；而使用的 SELinux 上下文类型是`mysqld_port_t`。

如果您配置**mysqld**使用不同的 TCP `port`，您可能需要为新端口设置上下文。例如，为非默认端口（如端口 3307）定义 SELinux 上下文：

```sql
semanage port -a -t mysqld_port_t -p tcp 3307
```

确认端口已添加：

```sql
$> semanage port -l | grep mysqld
mysqld_port_t                  tcp      3307, 1186, 3306, 63132-63164
```
