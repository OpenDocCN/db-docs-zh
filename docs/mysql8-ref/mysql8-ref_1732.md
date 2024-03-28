> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-add-node-example.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-add-node-example.html)

#### 25.6.7.3 在线添加 NDB 集群数据节点：详细示例

在本节中，我们提供了一个详细的示例，说明如何在线添加新的 NDB 集群数据节点，从一个单节点组中有 2 个数据节点的 NDB 集群开始，最终形成一个有 2 个节点组中有 4 个数据节点的集群。

**开始配置。** 为了说明，我们假设一个最小配置，并且集群使用一个只包含以下信息的`config.ini`文件：

```sql
[ndbd default]
DataMemory = 100M
IndexMemory = 100M
NoOfReplicas = 2
DataDir = /usr/local/mysql/var/mysql-cluster

[ndbd]
Id = 1
HostName = 198.51.100.1

[ndbd]
Id = 2
HostName = 198.51.100.2

[mgm]
HostName = 198.51.100.10
Id = 10

[api]
Id=20
HostName = 198.51.100.20

[api]
Id=21
HostName = 198.51.100.21
```

注意

我们在数据节点 ID 和其他节点之间的序列中留下了一个间隔。这样可以更容易地在以后为新添加的数据节点分配尚未使用的��点 ID。

我们还假设您已经使用适当的命令行或`my.cnf`选项启动了集群，并且在管理客户端中运行`SHOW`会产生类似于这里显示的输出：

```sql
-- NDB Cluster -- Management Client --
ndb_mgm> SHOW
Connected to Management Server at: 198.51.100.10:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=1    @198.51.100.1  (8.0.35-ndb-8.0.35, Nodegroup: 0, *)
id=2    @198.51.100.2  (8.0.35-ndb-8.0.35, Nodegroup: 0)

[ndb_mgmd(MGM)] 1 node(s)
id=10   @198.51.100.10  (8.0.35-ndb-8.0.35)

[mysqld(API)]   2 node(s)
id=20   @198.51.100.20  (8.0.35-ndb-8.0.35)
id=21   @198.51.100.21  (8.0.35-ndb-8.0.35)
```

最后，我们假设集群包含一个单`NDBCLUSTER`表，如下所示创建：

```sql
USE n;

CREATE TABLE ips (
    id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    country_code CHAR(2) NOT NULL,
    type CHAR(4) NOT NULL,
    ip_address VARCHAR(15) NOT NULL,
    addresses BIGINT UNSIGNED DEFAULT NULL,
    date BIGINT UNSIGNED DEFAULT NULL
)   ENGINE NDBCLUSTER;
```

后面显示的内存使用情况和相关信息是在将大约 50000 行插入到这个表后生成的。

注意

在这个示例中，我们展示了单线程的**ndbd**用于数据节点进程。如果您使用多线程的**ndbmtd**")，则可以通过在后续步骤中出现的地方用**ndbmtd**")替换**ndbd**来应用此示例。

**步骤 1：更新配置文件。** 在文本编辑器中打开集群全局配置文件，并添加与 2 个新数据节点对应的`[ndbd]`部分。（我们给这些数据节点分配 ID 3 和 4，并假设它们分别在地址 198.51.100.3 和 198.51.100.4 的主机上运行。）添加新部分后，`config.ini`文件的内容应该如下所示，文件的添加部分以粗体显示：

```sql
[ndbd default]
DataMemory = 100M
IndexMemory = 100M
NoOfReplicas = 2
DataDir = /usr/local/mysql/var/mysql-cluster

[ndbd]
Id = 1
HostName = 198.51.100.1

[ndbd]
Id = 2
HostName = 198.51.100.2

[ndbd]
Id = 3
HostName = 198.51.100.3

[ndbd]
Id = 4
HostName = 198.51.100.4

[mgm]
HostName = 198.51.100.10
Id = 10

[api]
Id=20
HostName = 198.51.100.20

[api]
Id=21
HostName = 198.51.100.21
```

一旦您做出必要的更改，请保存文件。

**步骤 2：重新启动管理服务器。** 重新启动集群管理服务器需要您发出停止管理服务器和然后再次启动的单独命令，如下所示：

1.  使用管理客户端`STOP`命令停止管理服务器，如下所示：

    ```sql
    ndb_mgm> 10 STOP
    Node 10 has shut down.
    Disconnecting to allow Management Server to shutdown

    $>
    ```

1.  由于关闭管理服务器会导致管理客户端终止，您必须从系统 shell 中启动管理服务器。为简单起见，我们假设`config.ini`与管理服务器二进制文件位于同一目录中，但实际上，您必须提供正确的配置文件路径。您还必须提供`--reload`或`--initial`选项，以便管理服务器从文件而不是其配置缓存中读取新配置。如果您的 shell 当前目录也与管理服务器二进制文件所在的目录相同，则可以按照以下方式调用管理服务器：

    ```sql
    $> ndb_mgmd -f config.ini --reload
    2008-12-08 17:29:23 [MgmSrvr] INFO     -- NDB Cluster Management Server. 8.0.35-ndb-8.0.35
    2008-12-08 17:29:23 [MgmSrvr] INFO     -- Reading cluster configuration from 'config.ini'
    ```

在重新启动**ndb_mgm**进程后，在管理客户端中检查`SHOW`的输出，您现在应该看到类似于以下内容：

```sql
-- NDB Cluster -- Management Client --
ndb_mgm> SHOW
Connected to Management Server at: 198.51.100.10:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=1    @198.51.100.1  (8.0.35-ndb-8.0.35, Nodegroup: 0, *)
id=2    @198.51.100.2  (8.0.35-ndb-8.0.35, Nodegroup: 0)
id=3 (not connected, accepting connect from 198.51.100.3)
id=4 (not connected, accepting connect from 198.51.100.4)

[ndb_mgmd(MGM)] 1 node(s)
id=10   @198.51.100.10  (8.0.35-ndb-8.0.35)

[mysqld(API)]   2 node(s)
id=20   @198.51.100.20  (8.0.35-ndb-8.0.35)
id=21   @198.51.100.21  (8.0.35-ndb-8.0.35)
```

**步骤 3：对现有数据节点执行滚动重启。** 可以在集群管理客户端中完全通过使用`RESTART`命令来完成此步骤，如下所示：

```sql
ndb_mgm> 1 RESTART
Node 1: Node shutdown initiated
Node 1: Node shutdown completed, restarting, no start.
Node 1 is being restarted

ndb_mgm> Node 1: Start initiated (version 8.0.35)
Node 1: Started (version 8.0.35)

ndb_mgm> 2 RESTART
Node 2: Node shutdown initiated
Node 2: Node shutdown completed, restarting, no start.
Node 2 is being restarted

ndb_mgm> Node 2: Start initiated (version 8.0.35)

ndb_mgm> Node 2: Started (version 8.0.35)
```

重要

在发出每个`*`X`* RESTART`命令后，请等待管理客户端报告`Node *`X`*: Started (version ...)` *之后* 再继续进行任何操作。

通过检查**mysql**客户端中的`ndbinfo.nodes`表，您可以验证所有现有数据节点是否使用更新的配置重新启动。

**步骤 4：对所有集群 API 节点执行滚动重启。** 关闭并重新启动充当集群中 SQL 节点的每个 MySQL 服务器，使用**mysqladmin shutdown**后跟**mysqld_safe**（或另一个启动脚本）。这应该类似于下面显示的内容，其中*`password`*是给定 MySQL 服务器实例的 MySQL `root`密码：

```sql
$> mysqladmin -uroot -p*password* shutdown
081208 20:19:56 mysqld_safe mysqld from pid file
/usr/local/mysql/var/tonfisk.pid ended
$> mysqld_safe --ndbcluster --ndb-connectstring=198.51.100.10 &
081208 20:20:06 mysqld_safe Logging to '/usr/local/mysql/var/tonfisk.err'.
081208 20:20:06 mysqld_safe Starting mysqld daemon with databases
from /usr/local/mysql/var
```

当然，确切的输入和输出取决于 MySQL 在系统上的安装方式和位置，以及您选择以何种选项启动它（以及是否在`my.cnf`文件中指定了一些或所有这些选项）。

**步骤 5：执行新数据节点的初始启动。** 在每个新数据节点的主机上的系统 shell 中，按照这里显示的方式启动数据节点，使用`--initial`选项：

```sql
$> ndbd -c 198.51.100.10 --initial
```

注意

与重新启动现有数据节点的情况不同，您可以同时启动新数据节点；您无需等待一个启动完成后再启动另一个。

*在继续下一步之前，请等待新的数据节点都已启动*。一旦新的数据节点启动，您可以在管理客户端`SHOW`命令的输出中看到它们尚未属于任何节点组（如此处以粗体显示）：

```sql
ndb_mgm> SHOW
Connected to Management Server at: 198.51.100.10:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=1    @198.51.100.1  (8.0.35-ndb-8.0.35, Nodegroup: 0, *)
id=2    @198.51.100.2  (8.0.35-ndb-8.0.35, Nodegroup: 0)
id=3    @198.51.100.3  (8.0.35-ndb-8.0.35, no nodegroup)
id=4    @198.51.100.4  (8.0.35-ndb-8.0.35, no nodegroup)

[ndb_mgmd(MGM)] 1 node(s)
id=10   @198.51.100.10  (8.0.35-ndb-8.0.35)

[mysqld(API)]   2 node(s)
id=20   @198.51.100.20  (8.0.35-ndb-8.0.35)
id=21   @198.51.100.21  (8.0.35-ndb-8.0.35)
```

**步骤 6：创建新的节点组。** 您可以通过在集群管理客户端中发出`CREATE NODEGROUP`命令来执行此操作。此命令以逗号分隔的数据节点的节点 ID 列表作为参数，如下所示：

```sql
ndb_mgm> CREATE NODEGROUP 3,4
Nodegroup 1 created
```

再次执行`SHOW`，您可以验证数据节点 3 和 4 已加入新的节点组（再次以粗体显示）：

```sql
ndb_mgm> SHOW
Connected to Management Server at: 198.51.100.10:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=1    @198.51.100.1  (8.0.35-ndb-8.0.35, Nodegroup: 0, *)
id=2    @198.51.100.2  (8.0.35-ndb-8.0.35, Nodegroup: 0)
id=3    @198.51.100.3  (8.0.35-ndb-8.0.35, Nodegroup: 1)
id=4    @198.51.100.4  (8.0.35-ndb-8.0.35, Nodegroup: 1)

[ndb_mgmd(MGM)] 1 node(s)
id=10   @198.51.100.10  (8.0.35-ndb-8.0.35)

[mysqld(API)]   2 node(s)
id=20   @198.51.100.20  (8.0.35-ndb-8.0.35)
id=21   @198.51.100.21  (8.0.35-ndb-8.0.35)
```

**步骤 7：重新分配集群数据。** 创建节点组时，现有数据和索引不会自动分配给新节点组的数据节点，您可以通过在管理客户端中发出适当的`REPORT`命令来查看：

```sql
ndb_mgm> ALL REPORT MEMORY

Node 1: Data usage is 5%(177 32K pages of total 3200)
Node 1: Index usage is 0%(108 8K pages of total 12832)
Node 2: Data usage is 5%(177 32K pages of total 3200)
Node 2: Index usage is 0%(108 8K pages of total 12832)
Node 3: Data usage is 0%(0 32K pages of total 3200)
Node 3: Index usage is 0%(0 8K pages of total 12832)
Node 4: Data usage is 0%(0 32K pages of total 3200)
Node 4: Index usage is 0%(0 8K pages of total 12832)
```

通过使用**ndb_desc**的`-p`选项，使输出包含分区信息，您可以看到该表仍然仅使用 2 个分区（在输出的`Per partition info`部分中以粗体显示）：

```sql
$> ndb_desc -c 198.51.100.10 -d n ips -p
-- ips --
Version: 1
Fragment type: 9
K Value: 6
Min load factor: 78
Max load factor: 80
Temporary table: no
Number of attributes: 6
Number of primary keys: 1
Length of frm data: 340
Row Checksum: 1
Row GCI: 1
SingleUserMode: 0
ForceVarPart: 1
FragmentCount: 2
TableStatus: Retrieved
-- Attributes --
id Bigint PRIMARY KEY DISTRIBUTION KEY AT=FIXED ST=MEMORY AUTO_INCR
country_code Char(2;latin1_swedish_ci) NOT NULL AT=FIXED ST=MEMORY
type Char(4;latin1_swedish_ci) NOT NULL AT=FIXED ST=MEMORY
ip_address Varchar(15;latin1_swedish_ci) NOT NULL AT=SHORT_VAR ST=MEMORY
addresses Bigunsigned NULL AT=FIXED ST=MEMORY
date Bigunsigned NULL AT=FIXED ST=MEMORY

-- Indexes --
PRIMARY KEY(id) - UniqueHashIndex
PRIMARY(id) - OrderedIndex

-- Per partition info --
Partition   Row count   Commit count  Frag fixed memory   Frag varsized memory
0           26086       26086         1572864             557056
1           26329       26329         1605632             557056
```

您可以通过为每个`NDB`表执行`ALTER TABLE ... ALGORITHM=INPLACE, REORGANIZE PARTITION`语句在**mysql**客户端中使数据重新分布到所有数据节点。

重要提示

`ALTER TABLE ... ALGORITHM=INPLACE, REORGANIZE PARTITION`不适用于使用`MAX_ROWS`选项创建的表。相反，使用`ALTER TABLE ... ALGORITHM=INPLACE, MAX_ROWS=...`来重新组织这些表。

请注意，使用`MAX_ROWS`设置每个表的分区数已被弃用，您应该使用`PARTITION_BALANCE`代替；有关更多信息，请参阅 Section 15.1.20.12, “Setting NDB Comment Options”。

执行语句`ALTER TABLE ips ALGORITHM=INPLACE, REORGANIZE PARTITION`后，您可以使用**ndb_desc**查看该表的数据现在使用 4 个分区存储，如下所示（相关部分以粗体显示）：

```sql
$> ndb_desc -c 198.51.100.10 -d n ips -p
-- ips --
Version: 16777217
Fragment type: 9
K Value: 6
Min load factor: 78
Max load factor: 80
Temporary table: no
Number of attributes: 6
Number of primary keys: 1
Length of frm data: 341
Row Checksum: 1
Row GCI: 1
SingleUserMode: 0
ForceVarPart: 1
FragmentCount: 4
TableStatus: Retrieved
-- Attributes --
id Bigint PRIMARY KEY DISTRIBUTION KEY AT=FIXED ST=MEMORY AUTO_INCR
country_code Char(2;latin1_swedish_ci) NOT NULL AT=FIXED ST=MEMORY
type Char(4;latin1_swedish_ci) NOT NULL AT=FIXED ST=MEMORY
ip_address Varchar(15;latin1_swedish_ci) NOT NULL AT=SHORT_VAR ST=MEMORY
addresses Bigunsigned NULL AT=FIXED ST=MEMORY
date Bigunsigned NULL AT=FIXED ST=MEMORY

-- Indexes --
PRIMARY KEY(id) - UniqueHashIndex
PRIMARY(id) - OrderedIndex

-- Per partition info --
Partition   Row count   Commit count  Frag fixed memory   Frag varsized memory
0           12981       52296         1572864             557056
1           13236       52515         1605632             557056
2           13105       13105         819200              294912
3           13093       13093         819200              294912
```

注意

通常，[`ALTER TABLE *`table_name`* [ALGORITHM=INPLACE,] REORGANIZE PARTITION`](alter-table.html "15.1.9 ALTER TABLE Statement")用于具有显式分区的表创建新的分区方案时会使用分区标识符列表和一组分区定义。在这种情况下，将数据重新分布到新的 NDB Cluster 节点组是一个例外；在这种情况下使用时，`REORGANIZE PARTITION`后不跟随其他关键字或标识符。

更多信息，请参见第 15.1.9 节，“ALTER TABLE Statement”。

另外，对于每个表，`ALTER TABLE`语句后应该跟着一个`OPTIMIZE TABLE`来回收浪费的空间。您可以通过以下查询在信息模式`TABLES`表中获取所有`NDBCLUSTER`表的列表：

```sql
SELECT TABLE_SCHEMA, TABLE_NAME
    FROM INFORMATION_SCHEMA.TABLES
    WHERE ENGINE = 'NDBCLUSTER';
```

注意

对于 NDB Cluster 表，`INFORMATION_SCHEMA.TABLES.ENGINE`值始终为`NDBCLUSTER`，无论用于创建表的`CREATE TABLE`语句（或用于将现有表从不同存储引擎转换为 NDB Cluster 的`ALTER TABLE`语句）中是否在`ENGINE`选项中使用了`NDB`或`NDBCLUSTER`。

在执行这些语句后，您可以在`ALL REPORT MEMORY`的输出中看到数据和索引现在在所有集群数据节点之间重新分布，如下所示：

```sql
ndb_mgm> ALL REPORT MEMORY

Node 1: Data usage is 5%(176 32K pages of total 3200)
Node 1: Index usage is 0%(76 8K pages of total 12832)
Node 2: Data usage is 5%(176 32K pages of total 3200)
Node 2: Index usage is 0%(76 8K pages of total 12832)
Node 3: Data usage is 2%(80 32K pages of total 3200)
Node 3: Index usage is 0%(51 8K pages of total 12832)
Node 4: Data usage is 2%(80 32K pages of total 3200)
Node 4: Index usage is 0%(50 8K pages of total 12832)
```

注意

由于在`NDBCLUSTER`表上只能执行一个 DDL 操作，您必须等待每个`ALTER TABLE ... REORGANIZE PARTITION`语句完成后再发出下一个。

对于在添加新数据节点之后创建的`NDBCLUSTER`表，不需要发出`ALTER TABLE ... REORGANIZE PARTITION`语句；添加到这些表中的数据会自动分布在所有数据节点之间。然而，在添加新节点之前存在的`NDBCLUSTER`表中，无论是现有数据还是新数据都不会使用新节点进行分布，直到使用`ALTER TABLE ... REORGANIZE PARTITION`重新组织这些表。

**另一种过程，无需滚动重启。** 可以通过在首次启动集群时配置额外的数据节点（但不启动它们）来避免需要滚动重启的需要。我们假设，与之前一样，您希望从两个数据节点（节点 1 和 2）开始，然后通过添加由节点 3 和 4 组成的第二个节点组来将集群扩展到四个数据节点：

```sql
[ndbd default]
DataMemory = 100M
IndexMemory = 100M
NoOfReplicas = 2
DataDir = /usr/local/mysql/var/mysql-cluster

[ndbd]
Id = 1
HostName = 198.51.100.1

[ndbd]
Id = 2
HostName = 198.51.100.2

[ndbd]
Id = 3
HostName = 198.51.100.3
Nodegroup = 65536

[ndbd]
Id = 4
HostName = 198.51.100.4
Nodegroup = 65536

[mgm]
HostName = 198.51.100.10
Id = 10

[api]
Id=20
HostName = 198.51.100.20

[api]
Id=21
HostName = 198.51.100.21
```

要稍后上线的数据节点（节点 3 和 4）可以配置为`NodeGroup = 65536`，在这种情况下，节点 1 和 2 可以按照这里所示启动：

```sql
$> ndbd -c 198.51.100.10 --initial
```

配置为`NodeGroup = 65536`的数据节点被管理服务器视为在等待由`StartNoNodeGroupTimeout`数据节点配置参数设置确定的一段时间后，使用`--nowait-nodes=3,4`启动节点 1 和 2。默认情况下，这是 15 秒（15000 毫秒）。

注意

`StartNoNodegroupTimeout` 必须对集群中的所有数据节点保持一致；因此，您应该始终在`config.ini`文件的`[ndbd default]`部分中设置它，而不是为单独的数据节点设置。

当您准备添加第二个节点组时，只需执行以下额外步骤：

1.  启动数据节点 3 和 4，为每个新节点调用一次数据节点进程：

    ```sql
    $> ndbd -c 198.51.100.10 --initial
    ```

1.  在管理客户端中发出适当的`CREATE NODEGROUP`命令：

    ```sql
    ndb_mgm> CREATE NODEGROUP 3,4
    ```

1.  在**mysql**客户端中，为每个现有的`NDBCLUSTER`表发出`ALTER TABLE ... REORGANIZE PARTITION`和`OPTIMIZE TABLE`语句。（正如本节其他地方所述，现有的 NDB Cluster 表在执行此操作之前无法使用新节点进行数据分发。）
