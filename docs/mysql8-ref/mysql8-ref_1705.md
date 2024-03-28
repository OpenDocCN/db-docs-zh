> 译文：[`dev.mysql.com/doc/refman/8.0/en/ndb-restore-different-number-nodes.html`](https://dev.mysql.com/doc/refman/8.0/en/ndb-restore-different-number-nodes.html)

#### 25.5.23.2 还原到不同数量的数据节点

可以将从 NDB 备份还原到具有不同数量数据节点的集群，与备份来源的原始集群数量不同。以下两个部分分别讨论了目标集群比备份源具有更少或更多数据节点的情况。

##### 25.5.23.2.1 还原到比原始节点少的节点

您可以将备份还原到比原始数据节点少的集群，只要较大数量的节点是较小数量的节点的偶数倍。在以下示例中，我们使用在具有四个数据节点的集群上进行的备份还原到具有两个数据节点的集群。

1.  原始集群的管理服务器位于主机`host10`上。原始集群有四个数据节点，节点 ID 和主机名如下所示，来自管理服务器的`config.ini`文件的摘录：

    ```sql
    [ndbd]
    NodeId=2
    HostName=host2

    [ndbd]
    NodeId=4
    HostName=host4

    [ndbd]
    NodeId=6
    HostName=host6

    [ndbd]
    NodeId=8
    HostName=host8
    ```

    我们假设每个数据节点最初都是使用**ndbmtd**") `--ndb-connectstring=host10`或等效方式启动的。

1.  以正常方式执行备份。有关如何执行此操作的信息，请参见 Section 25.6.8.2, “Using The NDB Cluster Management Client to Create a Backup”。

1.  每个数据节点备份创建的文件在这里列出，其中*`N`*是节点 ID，*`B`*是备份 ID。

    +   `BACKUP-*`B`*-0.*`N`*.Data`

    +   `BACKUP-*`B`*.*`N`*.ctl`

    +   `BACKUP-*`B`*.*`N`*.log`

    这些文件位于每个数据节点的`BackupDataDir``/BACKUP/BACKUP-*`B`*`下，在本示例的其余部分中，我们假设备份 ID 为 1。

    将所有这些文件保存以备将来复制到新数据节点（可以通过**ndb_restore**在数据节点的本地文件系统上访问）。将它们全部复制到一个位置最简单；我们假设您已经这样做了。

1.  目标集群的管理服务器位于主机`host20`上，目标集群有两个数据节点，节点 ID 和主机名如下所示，来自主机`host20`上的管理服务器`config.ini`文件：

    ```sql
    [ndbd]
    NodeId=3
    hostname=host3

    [ndbd]
    NodeId=5
    hostname=host5
    ```

    `host3`和`host5`上的每个数据节点进程应该使用**ndbmtd**") `-c host20` `--initial`或等效方式启动，以便新（目标）集群以干净的数据节点文件系统启动。

1.  将两组不同的两个备份文件复制到目标数据节点中的每一个。例如，将原始集群中节点 2 和 4 的备份文件复制到目标集群中的节点 3。这些文件如下所示：

    +   `BACKUP-1-0.2.Data`

    +   `BACKUP-1.2.ctl`

    +   `BACKUP-1.2.log`

    +   `BACKUP-1-0.4.Data`

    +   `BACKUP-1.4.ctl`

    +   `BACKUP-1.4.log`

    然后将节点 6 和 8 的备份文件复制到节点 5；这些文件如下列表所示：

    +   `BACKUP-1-0.6.Data`

    +   `BACKUP-1.6.ctl`

    +   `BACKUP-1.6.log`

    +   `BACKUP-1-0.8.Data`

    +   `BACKUP-1.8.ctl`

    +   `BACKUP-1.8.log`

    在本示例的其余部分中，我们假设相应的备份文件已保存在节点 3 和 5 上的目录`/BACKUP-1`中。

1.  在两个目标数据节点上，您必须从两组备份中恢复。首先，通过在`host3`上调用**ndb_restore**将节点 2 和 4 的备份还原到节点 3，如下所示：

    ```sql
    $> ndb_restore -c host20 --nodeid=2 --backupid=1 --restore-data --backup-path=/BACKUP-1

    $> ndb_restore -c host20 --nodeid=4 --backupid=1 --restore-data --backup-path=/BACKUP-1
    ```

    然后通过在`host5`上调用**ndb_restore**将节点 6 和 8 的备份还原到节点 5，如下所示：

    ```sql
    $> ndb_restore -c host20 --nodeid=6 --backupid=1 --restore-data --backup-path=/BACKUP-1

    $> ndb_restore -c host20 --nodeid=8 --backupid=1 --restore-data --backup-path=/BACKUP-1
    ```

##### 25.5.23.2.2 恢复到比原始节点更多的节点

给定**ndb_restore**命令指定的节点 ID 是原始备份中的节点 ID，而不是要将其恢复到的数据节点的节点 ID。当使用本节描述的方法执行备份时，**ndb_restore**连接到管理服务器并获取正在将备份恢复到的集群中的数据节点列表。恢复的数据相应地分布，因此在执行备份时不需要知道或计算目标集群中的节点数。

注意

当更改每个节点组的 LCP 线程或 LQH 线程的总数时，应重新创建使用**mysqldump**创建的备份的模式。

1.  *创建数据备份*。您可以通过从系统 shell 调用**ndb_mgm**客户端的`START BACKUP`命令来执行此操作，如下所示：

    ```sql
    $> ndb_mgm -e "START BACKUP 1"
    ```

    这假定所需的备份 ID 为 1。

1.  创建模式备份。只有在更改每个节点组的 LCP 线程或 LQH 线程的总数时才需要执行此步骤。

    ```sql
    $> mysqldump --no-data --routines --events --triggers --databases > myschema.sql
    ```

    重要

    一旦使用**ndb_mgm**创建了`NDB`本地备份，您在创建模式备份之前不得进行任何模式更改，如果这样做。

1.  将备份目录复制到新集群。例如，如果要恢复的备份具有 ID 1 和 `BackupDataDir` = `/backups/node_*`nodeid`*`，则此节点上备份的路径为 `/backups/node_1/BACKUP/BACKUP-1`。在此目录中有三个文件，列在此处：

    +   `BACKUP-1-0.1.Data`

    +   `BACKUP-1.1.ctl`

    +   `BACKUP-1.1.log`

    您应该将整个目录复制到新节点。

    如果您需要创建模式文件，请将其复制到 SQL 节点上的位置，以便**mysqld**可以读取。

没有要求必须从特定节点或节点恢复备份。

要从刚刚创建的备份中恢复，请执行以下步骤：

1.  *恢复模式*。

    +   如果您使用**mysqldump**创建了单独的模式备份文件，请使用**mysql**客户端导入此文件，类似于以下内容：

        ```sql
        $> mysql < myschema.sql
        ```

        在导入模式文件时，您可能需要指定 `--user` 和 `--password` 选项（可能还有其他选项），以便**mysql**客户端能够连接到 MySQL 服务器。

    +   如果您不需要创建模式文件，可以使用**ndb_restore** `--restore-meta`（简写为 `-m`）重新创建模式，类似于以下内容：

        ```sql
        $> ndb_restore --nodeid=1 --backupid=1 --restore-meta --backup-path=/backups/node_1/BACKUP/BACKUP-1
        ```

        **ndb_restore** 必须能够联系管理服务器；根据需要添加 `--ndb-connectstring` 选项以实现这一点。

1.  *恢复数据*。这需要针对原始集群中的每个数据节点执行一次，每次使用该数据节点的节点 ID。假设原始有 4 个数据节点，则所需的命令集将类似于以下内容：

    ```sql
    ndb_restore --nodeid=1 --backupid=1 --restore-data --backup-path=/backups/node_1/BACKUP/BACKUP-1 --disable-indexes
    ndb_restore --nodeid=2 --backupid=1 --restore-data --backup-path=/backups/node_2/BACKUP/BACKUP-1 --disable-indexes
    ndb_restore --nodeid=3 --backupid=1 --restore-data --backup-path=/backups/node_3/BACKUP/BACKUP-1 --disable-indexes
    ndb_restore --nodeid=4 --backupid=1 --restore-data --backup-path=/backups/node_4/BACKUP/BACKUP-1 --disable-indexes
    ```

    这些可以并行运行。

    请务必根据需要添加 `--ndb-connectstring` 选项。

1.  *重建索引*。这些索引已被禁用，因为在刚刚显示的命令中使用了 `--disable-indexes` 选项。重新创建索引可以避免由于恢复在所有点上不一致而导致的错误。重建索引在某些情况下还可以提高性能。要重建索引，请在单个节点上执行以下命令一次：

    ```sql
    $> ndb_restore --nodeid=1 --backupid=1 --backup-path=/backups/node_1/BACKUP/BACKUP-1 --rebuild-indexes
    ```

    正如之前提到的，您可能需要添加`--ndb-connectstring`选项，以便**ndb_restore**可以联系管理服务器。
