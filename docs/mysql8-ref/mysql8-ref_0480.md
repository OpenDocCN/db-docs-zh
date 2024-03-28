> 原文：[`dev.mysql.com/doc/refman/8.0/en/selinux-context-mysql-feature-ports.html`](https://dev.mysql.com/doc/refman/8.0/en/selinux-context-mysql-feature-ports.html)

#### 8.7.5.2 设置 MySQL 功能的 TCP 端口上下文

如果启用了某些 MySQL 功能，您可能需要为这些功能使用的额外端口设置 SELinux TCP 端口上下文。如果 MySQL 功能使用的端口没有正确的 SELinux 上下文，这些功能可能无法正常运行。

以下各节描述了如何为 MySQL 功能设置端口上下文。通常，可以使用相同的方法为任何 MySQL 功能设置端口上下文。有关 MySQL 功能使用的端口信息，请参考 MySQL 端口参考。

从 MySQL 8.0.14 到 MySQL 8.0.17，必须将 `mysql_connect_any` SELinux 布尔值设置为 `ON`。从 MySQL 8.0.18 开始，不再需要或建议启用 `mysql_connect_any`。

```sql
setsebool -P mysql_connect_any=ON
```

##### 设置 Group Replication 的 TCP 端口上下文

如果启用了 SELinux，您必须为 Group Replication 使用的通信端口设置端口上下文，该端口由 `group_replication_local_address` 变量定义。**mysqld** 必须能够绑定到 Group Replication 通信端口并在那里监听。InnoDB Cluster 依赖于 Group Replication，因此这同样适用于集群中使用的实例。要查看当前由 MySQL 使用的端口，请执行：

```sql
semanage port -l | grep mysqld
```

假设 Group Replication 通信端口为 33061，请通过以下方式设置端口上下文：

```sql
semanage port -a -t mysqld_port_t -p tcp 33061
```

##### 设置 Document Store 的 TCP 端口上下文

如果启用了 SELinux，您必须为 X Plugin 使用的通信端口设置端口上下文，该端口由 `mysqlx_port` 变量定义。**mysqld** 必须能够绑定到 X Plugin 通信端口并在那里监听。

假设 X Plugin 通信端口为 33060，请通过以下方式设置端口上下文：

```sql
semanage port -a -t mysqld_port_t -p tcp 33060
```

##### 设置 MySQL Router 的 TCP 端口上下文

如果启用了 SELinux，您必须为 MySQL Router 使用的通信端口设置端口上下文。假设 MySQL Router 使用的额外通信端口是默认的 6446、6447、64460 和 64470，在每个实例上通过以下方式设置端口上下文：

```sql
semanage port -a -t mysqld_port_t -p tcp 6446
semanage port -a -t mysqld_port_t -p tcp 6447
semanage port -a -t mysqld_port_t -p tcp 64460
semanage port -a -t mysqld_port_t -p tcp 64470
```
