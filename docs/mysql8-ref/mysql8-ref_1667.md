> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-connection-strings.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-connection-strings.html)

#### 25.4.3.3 NDB Cluster 连接字符串

除了 NDB Cluster 管理服务器（**ndb_mgmd**）之外，作为 NDB Cluster 的一部分的每个节点都需要一个指向管理服务器位置的连接字符串。此连接字符串用于建立与管理服务器的连接以及根据节点在集群中的角色执行其他任务。连接字符串的语法如下：

```sql
[nodeid=*node_id*, ]*host-definition*[, *host-definition*[, ...]]

*host-definition*:
    *host_name*[:*port_number*]
```

`node_id` 是一个大于或等于 1 的整数，用于标识 `config.ini` 中的一个节点。*`host_name`* 是一个表示有效的互联网主机名或 IP 地址的字符串。*`port_number`* 是一个指向 TCP/IP 端口号的整数。

```sql
example 1 (long):    "nodeid=2,myhost1:1100,myhost2:1100,198.51.100.3:1200"
example 2 (short):   "myhost1"
```

如果连接字符串中未提供任何值，则`localhost:1186`将用作默认连接字符串值。如果连接字符串中省略了*`port_num`*，则默认端口为 1186。该端口应始终在网络上可用，因为它已被 IANA 分配给此目的（有关详细信息，请参见[`www.iana.org/assignments/port-numbers`](http://www.iana.org/assignments/port-numbers)）。

通过列出多个主机定义，可以指定几个冗余的管理服务器。NDB Cluster 数据或 API 节点尝试按照指定顺序在每个主机上联系连续的管理服务器，直到建立成功的连接。

还可以在连接字符串中指定一个或多个绑定地址，以供具有多个网络接口的节点用于连接到管理服务器。绑定地址由主机名或网络地址和可选端口号组成。连接字符串的增强语法如下所示：

```sql
[nodeid=*node_id*, ]
    [bind-address=*host-definition*, ]
    *host-definition*[; bind-address=*host-definition*]
    *host-definition*[; bind-address=*host-definition*]
    [, ...]]

*host-definition*:
    *host_name*[:*port_number*]
```

如果在指定任何管理主机之前在连接字符串中使用单个绑定地址，则此地址将用作连接到任何管理主机的默认地址（除非针对给定管理服务器进行了覆盖；请参见本节后面的示例）。例如，以下连接字符串使节点使用 `198.51.100.242`，无论连接到哪个管理服务器：

```sql
bind-address=198.51.100.242, poseidon:1186, perch:1186
```

如果在管理主机定义之后指定了绑定地址，则仅用于连接到该管理节点。考虑以下连接字符串：

```sql
poseidon:1186;bind-address=localhost, perch:1186;bind-address=198.51.100.242
```

在这种情况下，节点使用 `localhost` 连接到运行在名为 `poseidon` 的主机上的管理服务器，使用 `198.51.100.242` 连接到运行在名为 `perch` 的主机上的管理服务器。

您可以指定默认绑定地址，然后为一个或多个特定的管理主机覆盖此默认值。在以下示例中，`localhost`用于连接运行在主机`poseidon`上的管理服务器；由于`198.51.100.242`被首先指定（在任何管理服务器定义之前），它是默认绑定地址，因此用于连接到主机`perch`和`orca`上的管理服务器：

```sql
bind-address=198.51.100.242,poseidon:1186;bind-address=localhost,perch:1186,orca:2200
```

有多种不同的方法来指定连接字符串：

+   每个可执行文件都有自己的命令行选项，可以在启动时指定管理服务器。（请参阅各个可执行文件的文档。）

+   也可以通过将其放置在管理服务器的`my.cnf`文件中的`[mysql_cluster]`部分中，一次性为集群中的所有节点设置连接字符串。

+   为了向后兼容，还有另外两个选项可用，使用相同的语法：

    1.  设置`NDB_CONNECTSTRING`环境变量以包含连接字符串。

    1.  将每个可执行文件的连接字符串写入名为`Ndb.cfg`的文本文件中，并将此文件放置在可执行文件的启动目录中。

    然而，这些现在已经被弃用，不应该用于新的安装。

指定连接字符串的推荐方法是在每个可执行文件的命令行或`my.cnf`文件中设置它。
