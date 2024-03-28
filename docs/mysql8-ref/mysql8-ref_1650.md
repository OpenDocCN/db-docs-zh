# 25.3.3 NDB 集群的初始配置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-configuration.html)

在本节中，我们讨论通过创建和编辑配置文件手动配置已安装的 NDB 集群。

对于我们的四节点、四主机 NDB 集群（请参见 Cluster nodes and host computers），需要编写四个配置文件，每个节点主机一个。

+   每个数据节点或 SQL 节点都需要一个提供两个信息的`my.cnf`文件：一个连接字符串，告诉节点在哪里找到管理节点，以及一行告诉此主机上的 MySQL 服务器（承载数据节点的机器）启用`NDBCLUSTER`存储引擎��

    有关连接字符串的更多信息，请参见 Section 25.4.3.3, “NDB Cluster Connection Strings”。

+   管理节点需要一个`config.ini`文件，告诉它要维护多少片段副本，为每个数据节点分配多少内存用于数据和索引，数据节点在哪里，每个数据节点在磁盘上保存数据的位置，以及如何找到任何 SQL 节点。

**配置数据节点和 SQL 节点。** 用于数据节点的`my.cnf`文件相当简单。配置文件应位于`/etc`目录中，并且可以使用任何文本编辑器进行编辑（如果文件不存在，则创建文件）。例如：

```sql
$> vi /etc/my.cnf
```

注意

我们在这里展示使用**vi**创建文件，但任何文本编辑器都可以正常工作。

对于我们示例设置中的每个数据节点和 SQL 节点，`my.cnf`应该如下所示：

```sql
[mysqld]
# Options for mysqld process:
ndbcluster                      # run NDB storage engine

[mysql_cluster]
# Options for NDB Cluster processes:
ndb-connectstring=198.51.100.10 # location of management server
```

输入上述信息后，保存此文件并退出文本编辑器。对承载数据节点“A”、数据节点“B”和 SQL 节点的机器执行此操作。

重要

一旦您在`my.cnf`文件的`[mysqld]`和`[mysql_cluster]`部分中使用了`ndbcluster`和`ndb-connectstring`参数启动了一个**mysqld**进程，您就不能在实际启动集群之前执行任何`CREATE TABLE`或`ALTER TABLE`语句。否则，这些语句将失败并显示错误。这是设计上的限制。

**配置管理节点。** 配置管理节点的第一步是创建包含配置文件的目录，然后创建文件本身。例如（以`root`身份运行）：

```sql
$> mkdir /var/lib/mysql-cluster
$> cd /var/lib/mysql-cluster
$> vi config.ini
```

对于我们的代表性设置，`config.ini`文件应该如下所示：

```sql
[ndbd default]
# Options affecting ndbd processes on all data nodes:
NoOfReplicas=2 # Number of fragment replicas
DataMemory=98M # How much memory to allocate for data storage

[ndb_mgmd]
# Management process options:
HostName=198.51.100.10 # Hostname or IP address of management node
DataDir=/var/lib/mysql-cluster # Directory for management node log files

[ndbd]
# Options for data node "A":
 # (one [ndbd] section per data node)
HostName=198.51.100.30 # Hostname or IP address
NodeId=2 # Node ID for this data node
DataDir=/usr/local/mysql/data # Directory for this data node's data files

[ndbd]
# Options for data node "B":
HostName=198.51.100.40 # Hostname or IP address
NodeId=3 # Node ID for this data node
DataDir=/usr/local/mysql/data # Directory for this data node's data files

[mysqld]
# SQL node options:
HostName=198.51.100.20 # Hostname or IP address
 # (additional mysqld connections can be
 # specified for this node for various
 # purposes such as running ndb_restore)
```

注意

`world`数据库可从`dev.mysql.com/doc/index-other.html`下载。

当所有配置文件都已创建并指定了这些最小选项后，您可以开始启动集群并验证所有进程是否正在运行。我们将在第 25.3.4 节，“NDB 集群的初始启动”中讨论如何执行此操作。

关于可用的 NDB 集群配置参数及其用途的更详细信息，请参见第 25.4.3 节，“NDB 集群配置文件”，以及第 25.4 节，“NDB 集群配置”。关于 NDB 集群配置与备份相关的内容，请参见第 25.6.8.3 节，“NDB 集群备份配置”。

注意

集群管理节点的默认端口是 1186；数据节点的默认端口是 2202。然而，集群可以自动为数据节点分配已经空闲的端口。
