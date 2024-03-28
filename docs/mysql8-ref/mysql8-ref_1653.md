# 25.3.6 安全关闭和重启 NDB 集群

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-shutdown-restart.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-shutdown-restart.html)

要关闭集群，请在托管管理节点的机器上的 shell 中输入以下命令：

```sql
$> ndb_mgm -e shutdown
```

这里的 `-e` 选项用于从 shell 传递命令给**ndb_mgm**客户端。该命令导致**ndb_mgm**、**ndb_mgmd**以及任何**ndbd**或**ndbmtd**")进程优雅地终止。任何 SQL 节点都可以使用**mysqladmin shutdown**和其他方法终止。在 Windows 平台上，假设您已将 SQL 节点安装为 Windows 服务，您可以使用 **SC STOP *`service_name`*** 或 **NET STOP *`service_name`***。

要在 Unix 平台上重新启动集群，请运行以下命令：

+   在管理主机（我们示例设置中的`198.51.100.10`）上：

    ```sql
    $> ndb_mgmd -f /var/lib/mysql-cluster/config.ini
    ```

+   在每个数据节点主机（`198.51.100.30`和`198.51.100.40`）上：

    ```sql
    $> ndbd
    ```

+   使用**ndb_mgm**客户端验证两个数据节点已成功启动。

+   在 SQL 主机（`198.51.100.20`）上：

    ```sql
    $> mysqld_safe &
    ```

在 Windows 平台上，假设您已使用默认服务名称将所有 NDB 集群进程安装为 Windows 服务（参见 Section 25.3.2.4, “Installing NDB Cluster Processes as Windows Services”)，您可以按以下方式重新启动集群：

+   在管理主机（我们示例设置中的`198.51.100.10`）上执行以下命令：

    ```sql
    C:\> SC START ndb_mgmd
    ```

+   在每个数据节点主机（`198.51.100.30`和`198.51.100.40`）上执行以下命令：

    ```sql
    C:\> SC START ndbd
    ```

+   在管理节点主机上，使用**ndb_mgm**客户端验证管理节点和两个数据节点已成功启动（参见 Section 25.3.2.3, “Initial Startup of NDB Cluster on Windows”）。

+   在 SQL 节点主机（`198.51.100.20`）上执行以下命令：

    ```sql
    C:\> SC START mysql
    ```

在生产环境中，通常不希望完全关闭集群。在许多情况下，即使进行配置更改或对集群硬件或软件（或两者）进行升级需要关闭单个主机，也可以通过对集群进行滚动重启而不必完全关闭集群来实现。有关如何执行此操作的更多信息，请参见第 25.6.5 节，“执行 NDB 集群的滚动重启”。
