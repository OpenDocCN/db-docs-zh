# 25.6.1 NDB Cluster 管理客户端中的命令

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-mgm-client-commands.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-mgm-client-commands.html)

除了中央配置文件外，集群还可以通过管理客户端提供的命令行界面进行控制 **ndb_mgm**。这是运行集群的主要管理界面。

事件日志的命令在 第 25.6.3 节，“NDB Cluster 生成的事件报告” 中给出；创建备份并从中恢复的命令在 第 25.6.8 节，“NDB Cluster 的在线备份” 中提供。

**使用 MySQL Cluster Manager 的 ndb_mgm。** MySQL Cluster Manager 1.4.8 提供对 NDB 8.0 的实验性支持。MySQL Cluster Manager 处理启动和停止进程，并在内部跟踪其状态，因此不需要使用 **ndb_mgm** 来执行这些任务，对于由 MySQL Cluster Manager 控制的 NDB Cluster。建议*不要*使用随 NDB Cluster 发行版一起提供的 **ndb_mgm** 命令行客户端执行涉及启动或停止节点的操作。这些操作包括但不限于 `START`、`STOP`、`RESTART` 和 `SHUTDOWN` 命令。更多信息，请参阅 MySQL Cluster Manager 进程命令。

管理客户端具有以下基本命令。在接下来的清单中，*`node_id`* 表示数据节点 ID 或关键字 `ALL`，表示该命令应用于集群的所有数据节点。

+   `CONNECT *`connection-string`*`

    连接到连接字符串指示的管理服务器。如果客户端已连接到此服务器，则客户端重新连接。

+   [`CREATE NODEGROUP *`nodeid`*[, *`nodeid`*, ...]`](mysql-cluster-mgm-client-commands.html#ndbclient-create-nodegroup)

    创建新的 NDB Cluster 节点组，并使数据节点加入其中。

    此命令用于在线添加新数据节点到 NDB 集群后，使它们加入新的节点组，从而开始完全参与集群。该命令的唯一参数是一个逗号分隔的节点 ID 列表—这些是刚刚添加并启动的节点的 ID，它们将加入新的节点组。列表中不能包含重复的 ID；从 NDB 8.0.26 开始，任何重复的存在会导致命令返回错误。列表中节点的数量必须与已经是集群一部分的每个节点组中的节点数量相同（每个 NDB 集群节点组必须有相同数量的节点）。换句话说，如果 NDB 集群由 2 个每个有 2 个数据节点的节点组组成，则新的节点组也必须有 2 个数据节点。

    由此命令创建的新节点组的节点组 ID 是自动确定的，并且始终是集群中未使用的最高的节点组 ID；无法手动设置它。

    欲了解更多信息，请参阅 第 25.6.7 节，“在线添加 NDB 集群数据节点”。

+   `DROP NODEGROUP *`nodegroup_id`*`

    删除具有给定 *`nodegroup_id`* 的 NDB 集群节点组。

    此命令可用于从 NDB 集群中删除节点组。`DROP NODEGROUP` 的唯一参数是要删除的节点��的节点组 ID。

    `DROP NODEGROUP` 仅用于从受影响的节点组中移除数据节点。它不会停止数据节点，也不会将它们分配给不同的节点组，或从集群的配置中移除它们。在管理客户端 `SHOW` 命令的输出中，不属于任何节点组的数据节点会显示为 `no nodegroup`，而不是节点组 ID，就像这样（使用粗体文本表示）：

    ```sql
    id=3    @10.100.2.67  (8.0.35-ndb-8.0.35, no nodegroup)
    ```

    `DROP NODEGROUP` 仅在欲删除的节点组中的所有数据节点完全不包含任何表数据和表定义时才有效。由于目前无法使用 **ndb_mgm** 或 **mysql** 客户端从特定数据节点或节点组中删除所有数据，这意味着该命令仅在以下两种情况下成功：

    1.  在 **ndb_mgm** 客户端中发出 `CREATE NODEGROUP` 命令之后，但在 **mysql** 客户端中发出任何 `ALTER TABLE ... REORGANIZE PARTITION` 命令之前。

    1.  在使用`DROP TABLE`删除所有`NDBCLUSTER`表之后。

        `TRUNCATE TABLE`对此目的无效，因为这只会删除表数据；数据节点将继续存储`NDBCLUSTER`表的定义，直到发出导致表元数据被删除的`DROP TABLE`语句。

    有关`DROP NODEGROUP`的更多信息，请参见 Section 25.6.7, “Adding NDB Cluster Data Nodes Online”。

+   `ENTER SINGLE USER MODE *`node_id`*`

    进入单用户模式，只允许由节点 ID *`node_id`* 标识的 MySQL 服务器访问数据库。

    **ndb_mgm**客户端明确确认已发出此命令并已生效，如下所示：

    ```sql
    ndb_mgm> ENTER SINGLE USER MODE 100
    Single user mode entered
    Access is granted for API node 100 only.
    ```

    此外，在单用户模式下具有独占访问权限的 API 或 SQL 节点在`SHOW`命令的输出中会被指示，如下所示：

    ```sql
    ndb_mgm> SHOW
    Cluster Configuration
    ---------------------
    [ndbd(NDB)]     2 node(s)
    id=5    @127.0.0.1  (mysql-8.0.35 ndb-8.0.35, single user mode, Nodegroup: 0, *)
    id=6    @127.0.0.1  (mysql-8.0.35 ndb-8.0.35, single user mode, Nodegroup: 0)

    [ndb_mgmd(MGM)] 1 node(s)
    id=50   @127.0.0.1  (mysql-8.0.35 ndb-8.0.35)

    [mysqld(API)]   2 node(s)
    id=100  @127.0.0.1  (mysql-8.0.35 ndb-8.0.35, allowed single user)
    id=101 (not connected, accepting connect from any host)
    ```

+   `EXIT SINGLE USER MODE`

    退出单用户模式，使所有 SQL 节点（即所有运行的**mysqld**进程）可以访问数据库。

    注意

    即使不在单用户模式下，也可以使用`EXIT SINGLE USER MODE`，但在这种情况下该命令不起作用。

+   `HELP`

    显示所有可用命令的信息。

+   `*`node_id`* NODELOG DEBUG {ON|OFF}`

    在节点日志中切换调试日志记录，就好像受影响的数据节点已经使用`--verbose`选项启动一样。`NODELOG DEBUG ON`开始调试日志记录；`NODELOG DEBUG OFF`关闭调试日志记录。

+   [`PROMPT [*`prompt`*]`](mysql-cluster-mgm-client-commands.html#ndbclient-prompt)

    更改**ndb_mgm**显示的提示为字符串文字 *`prompt`*。

    *`prompt`* 不应该被引用（除非您希望提示包含引号）。与**mysql**客户端不同，不会识别特殊字符序列和转义。如果没有参数调用，该命令将重置提示为默认值（`ndb_mgm>`）。

    ��下是一些示例：

    ```sql
    ndb_mgm> PROMPT mgm#1:
    mgm#1: SHOW
    Cluster Configuration
    ...
    mgm#1: PROMPT mymgm >
    mymgm > PROMPT 'mymgm:'
    'mymgm:' PROMPT  mymgm:
    mymgm: PROMPT
    ndb_mgm> EXIT
    $>
    ```

    请注意，*`prompt`*字符串中的前导空格和空格不会被修剪。尾随空格会被移除。

+   `QUIT`, `EXIT`

    终止管理客户端。

    此命令不会影响连接到集群的任何节点。

+   `*`node_id`* REPORT *`report-type`*`

    显示标识为*`node_id`*的数据节点或使用`ALL`显示的*`report-type`*类型报告。

    目前，*`report-type`*有三个可接受的值：

    +   `BackupStatus` 提供正在进行的集群备份的状态报告

    +   `MemoryUsage` 显示每个数据节点使用的数据内存和索引内存量，如下所示：

        ```sql
        ndb_mgm> ALL REPORT MEMORY

        Node 1: Data usage is 5%(177 32K pages of total 3200)
        Node 1: Index usage is 0%(108 8K pages of total 12832)
        Node 2: Data usage is 5%(177 32K pages of total 3200)
        Node 2: Index usage is 0%(108 8K pages of total 12832)
        ```

        这些信息也可以从`ndbinfo.memoryusage`表中获取。

    +   `EventLog` 报告一个或多个数据节点的事件日志缓冲区中的事件。

    *`report-type`*对大小写不敏感且“模糊”；对于`MemoryUsage`，可以使用`MEMORY`（如前面的示例中所示），`memory`，甚至简单地使用`MEM`（或`mem`）。类似地，您可以缩写`BackupStatus`。

+   [`*`node_id`* RESTART [-n] [-i] [-a] [-f]`](mysql-cluster-mgm-client-commands.html#ndbclient-restart)

    重新启动标识为*`node_id`*（或所有数据节点）的数据节点。

    使用`RESTART`选项与`-i`一起会导致数据节点执行初始重启；也就是说，节点的文件系统会被删除并重新创建。效果与停止数据节点进程，然后在系统 shell 中使用**ndbd** `--initial`重新启动相同。

    注意

    当使用此选项时，备份文件和磁盘数据文件不会被删除。

    使用`-n`选项会导致数据节点进程重新启动，但数据节点实际上不会上线，直到发出适当的`START`命令为止。此选项的效果与停止数据节点，然后在系统 shell 中使用**ndbd** `--nostart`或**ndbd** `-n`重新启动相同。

    使用`-a`会导致依赖于此节点的所有当前事务被中止。当节点重新加入集群时，不会执行 GCP 检查。

    通常，如果使节点脱机会导致集群不完整，则`RESTART`会失败。`-f`选项会强制节点在不检查此情况下重新启动。如果使用此选项并导致集群不完整，则整个集群会重新启动。

+   `SHOW`

    显示有关集群和集群节点的基本信息。对于所有节点，输出包括节点的 ID、类型和`NDB`软件版本。如果节点已连接，还会显示其 IP 地址；否则输出显示`not connected, accepting connect from *`ip_address`*`，对于允许从任何地址连接的节点，使用`any host`。

    此外，对于数据节点，如果节点尚未启动，则输出包括`starting`，并显示节点所属的节点组。如果数据节点充当主节点，则用星号（`*`）表示。

    考虑一个配置文件包含此处显示的信息的集群（为清晰起见，可能的其他设置被省略）：

    ```sql
    [ndbd default]
    DataMemory= 128G
    NoOfReplicas= 2

    [ndb_mgmd]
    NodeId=50
    HostName=198.51.100.150

    [ndbd]
    NodeId=5
    HostName=198.51.100.10
    DataDir=/var/lib/mysql-cluster

    [ndbd]
    NodeId=6
    HostName=198.51.100.20
    DataDir=/var/lib/mysql-cluster

    [ndbd]
    NodeId=7
    HostName=198.51.100.30
    DataDir=/var/lib/mysql-cluster

    [ndbd]
    NodeId=8
    HostName=198.51.100.40
    DataDir=/var/lib/mysql-cluster

    [mysqld]
    NodeId=100
    HostName=198.51.100.100

    [api]
    NodeId=101
    ```

    在启动此集群（包括一个 SQL 节点）后，`SHOW`显示以下输出：

    ```sql
    ndb_mgm> SHOW
    Connected to Management Server at: localhost:1186
    Cluster Configuration
    ---------------------
    [ndbd(NDB)]     4 node(s)
    id=5    @198.51.100.10  (mysql-8.0.35 ndb-8.0.35, Nodegroup: 0, *)
    id=6    @198.51.100.20  (mysql-8.0.35 ndb-8.0.35, Nodegroup: 0)
    id=7    @198.51.100.30  (mysql-8.0.35 ndb-8.0.35, Nodegroup: 1)
    id=8    @198.51.100.40  (mysql-8.0.35 ndb-8.0.35, Nodegroup: 1)

    [ndb_mgmd(MGM)] 1 node(s)
    id=50   @198.51.100.150  (mysql-8.0.35 ndb-8.0.35)

    [mysqld(API)]   2 node(s)
    id=100  @198.51.100.100  (mysql-8.0.35 ndb-8.0.35)
    id=101 (not connected, accepting connect from any host)
    ```

    此命令的输出还指示集群何时处于单用户模式（请参阅`进入单用户模式`命令的描述，以及第 25.6.6 节，“NDB 集群单用户模式”）。在 NDB 8.0 中，它还指示在此模式生效时哪个 API 或 SQL 节点具有独占访问权限；这仅在所有连接到集群的数据节点和管理节点运行 NDB 8.0 时有效。

+   `SHUTDOWN`

    关闭所有集群数据节点和管理节点。在执行此操作后退出管理客户端，请使用`EXIT`或`QUIT`。

    此命令*不会*关闭连接到集群的任何 SQL 节点或 API 节点。

+   `*`node_id`*启动`

    将由*`node_id`*标识的数据节点（或所有数据节点）在线化。

    `ALL START`仅对所有数据节点起作用，不影响管理节点。

    重要

    要使用此命令将数据节点在线化，数据节点必须使用`--nostart`或`-n`启动。

+   `*`node_id`*状态`

    显示由*`node_id`*标识的数据节点（或所有数据节点）的状态信息。

    可能的节点状态值包括`UNKNOWN`、`NO_CONTACT`、`NOT_STARTED`、`STARTING`、`STARTED`、`SHUTTING_DOWN`和`RESTARTING`。

    此命令的输出还指示集群何时处于单用户模式。

+   [`*`node_id`* STOP [-a] [-f]`](mysql-cluster-mgm-client-commands.html#ndbclient-stop)

    停止由*`node_id`*标识的数据或管理节点。

    注意

    `ALL STOP`仅用于停止所有数据节点，不影响管理节点。

    受此命令影响的节点会从集群中断开连接，并且其关联的**ndbd**或**ndb_mgmd**进程终止。

    `-a`选项会导致节点立即停止，而不等待任何待处理事务的完成。

    通常，如果结果导致集群不完整，则`STOP`会失败。`-f`选项强制节点关闭而不检查此情况。如果使用此选项且结果是不完整的集群，则集群立即关闭。

    警告

    使用`-a`选项还会禁用通常在调用`STOP`时执行的安全检查，以确保停止节点不会导致集群不完整。换句话说，当使用`STOP`命令时，使用`-a`选项时应该非常小心，因为该选项使得集群可能会因为不再具有存储在`NDB`中的所有数据的完整副本而被强制关闭。

**附加命令。** **ndb_mgm**客户端中提供了其他一些命令，如下列表所示：

+   `START BACKUP`用于在**ndb_mgm**客户端中执行在线备份；`ABORT BACKUP`命令用于取消已经进行中的备份。有关更多信息，请参阅第 25.6.8 节，“NDB 集群的在线备份”。

+   `CLUSTERLOG`命令用于执行各种日志功能。有关更多信息和示例，请参阅第 25.6.3 节，“NDB 集群生成的事件报告”。`NODELOG DEBUG`激活或停用节点日志中的调试打印输出，如本节先前所述。

+   用于测试和诊断工作，客户端支持一个`DUMP`命令，可用于在集群上执行内部命令。除非由 MySQL 支持指示，否则不应在生产环境中使用。有关更多信息，请参阅 NDB 集群管理客户端 DUMP 命令。
