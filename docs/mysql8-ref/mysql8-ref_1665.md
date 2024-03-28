> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-config-example.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-config-example.html)

#### 25.4.3.1 NDB Cluster 配置：基本示例

要支持 NDB Cluster，您应该按照以下示例更新`my.cnf`。您也可以在调用可执行文件时在命令行上指定这些参数。

注意

此处显示的选项不应与在`config.ini`全局配置文件中使用的选项混淆。全局配置选项将在本节后面讨论。

```sql
# my.cnf
# example additions to my.cnf for NDB Cluster
# (valid in MySQL 8.0)

# enable ndbcluster storage engine, and provide connection string for
# management server host (default port is 1186)
[mysqld]
ndbcluster
ndb-connectstring=ndb_mgmd.mysql.com

# provide connection string for management server host (default port: 1186)
[ndbd]
connect-string=ndb_mgmd.mysql.com

# provide connection string for management server host (default port: 1186)
[ndb_mgm]
connect-string=ndb_mgmd.mysql.com

# provide location of cluster configuration file
# IMPORTANT: When starting the management server with this option in the
# configuration file, the use of --initial or --reload on the command line when
# invoking ndb_mgmd is also required.
[ndb_mgmd]
config-file=/etc/config.ini
```

（有关连接字符串的更多信息，请参见第 25.4.3.3 节，“NDB Cluster 连接字符串”。）

```sql
# my.cnf
# example additions to my.cnf for NDB Cluster
# (works on all versions)

# enable ndbcluster storage engine, and provide connection string for management
# server host to the default port 1186
[mysqld]
ndbcluster
ndb-connectstring=ndb_mgmd.mysql.com:1186
```

重要

一旦您在`my.cnf`文件中以前面显示的方式使用`NDBCLUSTER`和`ndb-connectstring`参数启动了一个**mysqld**进程，您就不能执行任何`CREATE TABLE`或`ALTER TABLE`语句，除非您实际启动了集群。否则，这些语句将因错误而失败。*这是设计如此*。

您还可以在集群`my.cnf`文件中使用单独的`[mysql_cluster]`部分来设置所有可执行文件读取和使用的设置：

```sql
# cluster-specific settings
[mysql_cluster]
ndb-connectstring=ndb_mgmd.mysql.com:1186
```

对于可以在`my.cnf`文件中设置的其他`NDB`变量，请参见第 25.4.3.9.2 节，“NDB Cluster 系统变量”。

NDB Cluster 全局配置文件按照惯例命名为`config.ini`（但这不是必需的）。如果需要，它会在启动时被**ndb_mgmd**读取，并且可以放置在任何可以被其读取的位置。配置的位置和名称使用`--config-file=*`path_name`*`在命令行上与**ndb_mgmd**指定。此选项没有默认值，并且如果**ndb_mgmd**使用配置缓存，则会被忽略。

NDB Cluster 的全局配置文件使用 INI 格式，由方括号括起的部分标题（前面有部分标题），后跟适当的参数名称和值组成。与标准 INI 格式的一个偏差是，参数名称和值可以用冒号（`:`）分隔，也可以用等号（`=`）分隔；但是，等号更受青睐。另一个偏差是，部分不是通过部分名称唯一标识的。相反，唯一部分（例如相同类型的两个不同节点）通过在部分内指定的唯一 ID 来标识。

大多数参数都定义了默认值，并且也可以在`config.ini`中指定。要创建默认值部分，只需在部分名称中添加单词`default`。例如，`[ndbd]`部分包含适用于特定数据节点的参数，而`[ndbd default]`部分包含适用于所有数据节点的参数。假设所有数据节点应使用相同的数据内存大小。要配置它们所有，创建一个包含`DataMemory`行以指定数据内存大小的`[ndbd default]`部分。

如果使用，`[ndbd default]`部分必须在配置文件中的任何`[ndbd]`部分之前。对于任何其他类型的`default`部分也是如此。

注意

在一些较旧的 NDB Cluster 版本中，没有`NoOfReplicas`的默认值，它总是必须在`[ndbd default]`部分中明确指定。尽管此参数现在具有默认值 2，这是大多数常见用法场景中推荐的设置，但仍建议显式设置此参数。

全局配置文件必须定义集群中涉及的计算机和节点，以及这些节点位于哪些计算机上。这里显示了一个由一个管理服务器、两个数据节点和两个 MySQL 服务器组成的集群的简单配置文件示例：

```sql
# file "config.ini" - 2 data nodes and 2 SQL nodes
# This file is placed in the startup directory of ndb_mgmd (the
# management server)
# The first MySQL Server can be started from any host. The second
# can be started only on the host mysqld_5.mysql.com

[ndbd default]
NoOfReplicas= 2
DataDir= /var/lib/mysql-cluster

[ndb_mgmd]
Hostname= ndb_mgmd.mysql.com
DataDir= /var/lib/mysql-cluster

[ndbd]
HostName= ndbd_2.mysql.com

[ndbd]
HostName= ndbd_3.mysql.com

[mysqld]
[mysqld]
HostName= mysqld_5.mysql.com
```

注意

前面的示例旨在作为熟悉 NDB Cluster 的最小起始配置，并且几乎肯定不足以用于生产设置。请参阅第 25.4.3.2 节，“NDB Cluster 的推荐起始配置”，其中提供了一个更完整的起始配置示例。

每个节点在`config.ini`文件中都有自己的部分。例如，此集群有两个数据节点，因此前面的配置文件包含定义这些节点的两个`[ndbd]`部分。

注意

不要在`config.ini`文件中的部分标题相同行上放置注释；这会导致管理服务器无法启动，因为在这种情况下无法解析配置文件。

##### config.ini 文件的部分

`config.ini`配置文件中有六个不同的部分可供使用，如下列表所述：

+   `[computer]`: 定义了集群主机。配置可行的 NDB Cluster 不需要此部分，但在设置大型集群时可能会作为一种便利。有关更多信息，请参见第 25.4.3.4 节，“在 NDB Cluster 中定义计算机”。

+   `[ndbd]`: 定义了集群数据节点（**ndbd**进程）。有关详细信息，请参见第 25.4.3.6 节，“定义 NDB Cluster 数据节点”。

+   `[mysqld]`: 定义了集群的 MySQL 服务器节点（也称为 SQL 或 API 节点）。有关 SQL 节点配置的讨论，请参见第 25.4.3.7 节，“在 NDB Cluster 中定义 SQL 和其他 API 节点”。

+   `[mgm]`或`[ndb_mgmd]`: 定义了集群管理服务器（MGM）节点。有关管理节点配置的信息，请参见第 25.4.3.5 节，“定义 NDB Cluster 管理服务器”。

+   `[tcp]`: 定义了集群节点之间的 TCP/IP 连接，TCP/IP 是默认的传输协议。通常情况下，设置 NDB Cluster 不需要`[tcp]`或`[tcp default]`部分，因为集群会自动处理；但在某些情况下可能需要覆盖集群提供的默认值。有关可用 TCP/IP 配置参数及其使用方法，请参见第 25.4.3.10 节，“NDB Cluster TCP/IP Connections”。在某些情况下，您可能还会发现第 25.4.3.11 节，“NDB Cluster TCP/IP Connections Using Direct Connections”也很有用。

+   `[shm]`: 定义了节点之间的共享内存连接。在 MySQL 8.0 中，默认启用，但仍应被视为实验性。有关 SHM 互连的讨论，请参见第 25.4.3.12 节，“NDB Cluster Shared-Memory Connections”。

+   `[sci]`: 定义了集群数据节点之间的可扩展一致性接口连接。在 NDB 8.0 中不受支持。

您可以为每个部分定义`default`值。如果使用，`default`部分应该出现在该类型的任何其他部分之前。例如，`[ndbd default]`部分应该在任何`[ndbd]`部分之前出现在配置文件中。

NDB Cluster 参数名称不区分大小写，除非在 MySQL Server 的`my.cnf`或`my.ini`文件中指定。
