# 25.3.4 NDB Cluster 的初始启动

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-first-start.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-first-start.html)

在配置完成后，启动集群并不是很困难。每个集群节点进程必须分别在其所在的主机上启动。应首先启动管理节点，然后是数据节点，最后是任何 SQL 节点：

1.  在管理主机上，从系统 shell 发出以下命令以启动管理节点进程：

    ```sql
    $> ndb_mgmd --initial -f /var/lib/mysql-cluster/config.ini
    ```

    第一次启动时，必须告诉**ndb_mgmd**在哪里找到其配置文件，使用`-f`或`--config-file`选项。此选项要求还必须指定`--initial`或`--reload`；有关详细信息，请参阅第 25.5.4 节，“ndb_mgmd — The NDB Cluster Management Server Daemon”。

1.  在每个数据节点主机上运行以下命令以启动**ndbd**进程：

    ```sql
    $> ndbd
    ```

1.  如果您在 SQL 节点所在的集群主机上使用 RPM 文件安装 MySQL，则可以（也应该）使用提供的启动脚本来启动 SQL 节点上的 MySQL 服务器进程。

如果一切顺利，且集群已正确设置，那么集群现在应该是可操作的。您可以通过调用**ndb_mgm**管理节点客户端来测试。输出应该看起来像这里显示的那样，尽管根据您使用的 MySQL 确切版本的不同，输出可能会有一些细微差异：

```sql
$> ndb_mgm
-- NDB Cluster -- Management Client --
ndb_mgm> SHOW
Connected to Management Server at: localhost:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=2    @198.51.100.30  (Version: 8.0.35-ndb-8.0.35, Nodegroup: 0, *)
id=3    @198.51.100.40  (Version: 8.0.35-ndb-8.0.35, Nodegroup: 0)

[ndb_mgmd(MGM)] 1 node(s)
id=1    @198.51.100.10  (Version: 8.0.35-ndb-8.0.35)

[mysqld(API)]   1 node(s)
id=4    @198.51.100.20  (Version: 8.0.35-ndb-8.0.35)
```

此处将 SQL 节点称为`[mysqld(API)]`，这反映了**mysqld**进程充当 NDB Cluster API 节点的事实。

注意

在`SHOW`的输出中，给定 NDB Cluster SQL 或其他 API 节点的 IP 地址是 SQL 或 API 节点用于连接到集群数据节点的地址，而不是任何管理节点。

现在，您应该已经准备好在 NDB Cluster 中使用数据库、表格和数据了。请参阅第 25.3.5 节，“带有表格和数据的 NDB Cluster 示例”进行简要讨论。
