# 25.7.9 NDB 集群复制的 NDB 集群备份

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-backups.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-backups.html)

25.7.9.1 NDB 集群复制：自动同步副本到源二进制日志

25.7.9.2 使用 NDB 集群复制进行时点恢复

本节讨论使用 NDB 集群复制进行备份和从备份中恢复。我们假设复制服务器已经按照之前的说明进行了配置（请参阅第 25.7.5 节，“为 NDB 集群准备复制”，以及紧随其后的各节）。完成这些步骤后，进行备份然后从备份中恢复的过程如下：

1.  有两种不同的方法可以启动备份。

    +   **方法 A. ** 此方法要求在启动复制过程之前，在源服务器上先启用集群备份过程。可以通过在 `my.cnf` 文件中的 `[mysql_cluster]` 部分中包含以下行来完成，其中 *`management_host`* 是源集群的 `NDB` 管理服务器的 IP 地址或主机名，*`port`* 是管理服务器的端口号：

        ```sql
        ndb-connectstring=*management_host*[:*port*]
        ```

        注意

        只有在未使用默认端口（1186）时才需要指定端口号。有关 NDB 集群中端口和端口分配的更多信息，请参见第 25.3.3 节，“NDB 集群的初始配置”。

        在这种情况下，可以通过在复制源上执行此语句来启动备份：

        ```sql
        shell*S*> ndb_mgm -e "START BACKUP"
        ```

    +   **方法 B. ** 如果 `my.cnf` 文件未指定管理主机的位置，则可以通过将此信息作为 `START BACKUP` 命令的一部分传递给 `NDB` 管理客户端来启动备份过程。可以按照以下方式执行此操作，其中 *`management_host`* 和 *`port`* 是管理服务器的主机名和端口号：

        ```sql
        shell*S*> ndb_mgm *management_host*:*port* -e "START BACKUP"
        ```

        在我们之前概述的场景中（请参阅第 25.7.5 节，“为 NDB 集群准备复制”），将执行如下操作：

        ```sql
        shell*S*> ndb_mgm rep-source:1186 -e "START BACKUP"
        ```

1.  将集群备份文件复制到正在上线的复制品。运行源集群的每个为**ndbd**进程的系统上都有集群备份文件，*所有*这些文件都必须复制到复制品以确保成功恢复。备份文件可以复制到复制品的管理主机所在计算机上的任何目录中，只要 MySQL 和 NDB 二进制文件在该目录中具有读取权限。在这种情况下，我们假设这些文件已复制到目录`/var/BACKUPS/BACKUP-1`。

    虽然复制集群不必具有与源相同数量的数据节点，但强烈建议此数量相同。当复制服务器启动时，*必须*防止复制过程启动。您可以通过在命令行上使用`--skip-slave-start`选项启动复制品，通过在复制品的`my.cnf`文件中包含`skip-slave-start`，或在 NDB 8.0.24 或更高版本中，通过设置`skip_slave_start`系统变量来实现这一点。

1.  在复制品集群上创建任何在源集群上存在且需要复制的数据库。

    重要

    必须在复制品集群中的每个 SQL 节点上执行与要复制的每个数据库对应的`CREATE DATABASE`（或`CREATE SCHEMA`）语句。

1.  在**mysql**客户端中使用以下语句重置复制集群：

    ```sql
    mysql*R*> RESET SLAVE;
    ```

    在 NDB 8.0.22 或更高版本中，您还可以使用此语句：

    ```sql
    mysql*R*> RESET REPLICA;
    ```

1.  现在，您可以使用**ndb_restore**命令依次为每个备份文件启动集群恢复过程。对于其中的第一个，有必要包含`-m`选项以恢复集群元数据，如下所示：

    ```sql
    shell*R*> ndb_restore -c *replica_host*:*port* -n *node-id* \
            -b *backup-id* -m -r *dir*
    ```

    *`dir`*是备份文件在复制品上放置的目录路径。对于剩余备份文件对应的**ndb_restore**命令，不应使用`-m`选项。

    从具有四个数据节点的源集群中恢复（如第 25.7 节“NDB 集群复制”中所示的图表），其中备份文件已复制到目录`/var/BACKUPS/BACKUP-1`，在复制品上执行的正确命令序列可能如下所示：

    ```sql
    shell*R*> ndb_restore -c replica-host:1186 -n 2 -b 1 -m \
            -r ./var/BACKUPS/BACKUP-1
    shell*R*> ndb_restore -c replica-host:1186 -n 3 -b 1 \
            -r ./var/BACKUPS/BACKUP-1
    shell*R*> ndb_restore -c replica-host:1186 -n 4 -b 1 \
            -r ./var/BACKUPS/BACKUP-1
    shell*R*> ndb_restore -c replica-host:1186 -n 5 -b 1 -e \
            -r ./var/BACKUPS/BACKUP-1
    ```

    重要

    在这个例子中，**ndb_restore** 的最后调用中需要 `-e`（或 `--restore-epoch`）选项，以确保时代写入副本的 `mysql.ndb_apply_status` 表。没有这些信息，副本无法与源正确同步。 (参见 Section 25.5.23, “ndb_restore — Restore an NDB Cluster Backup”.)

1.  现在，您需要从副本的 `ndb_apply_status` 表中获取最新的时代（如 Section 25.7.8, “Implementing Failover with NDB Cluster Replication” 中所讨论的）：

    ```sql
    mysql*R*> SELECT @latest:=MAX(epoch)
            FROM mysql.ndb_apply_status;
    ```

1.  使用前一步骤中获得的 `@latest` 作为时代值，您可以从源的 `mysql.ndb_binlog_index` 表中获取正确的起始位置 `@pos` 和正确的二进制日志文件 `@file`。此处显示的查询从逻辑恢复位置之前应用的最后时代的 `Position` 和 `File` 列中获取这些值：

    ```sql
    mysql*S*> SELECT
     ->     @file:=SUBSTRING_INDEX(File, '/', -1),
     ->     @pos:=Position
     -> FROM mysql.ndb_binlog_index
     -> WHERE epoch > @latest
     -> ORDER BY epoch ASC LIMIT 1;
    ```

    如果当前没有复制流量，您可以通过在源上运行 `SHOW MASTER STATUS` 并使用输出中 `Position` 列中显示的值来获取类似信息，该值为所有文件中具有最大值后缀的文件的 `File` 列中显示的值。在这种情况下，您必须确定这是哪个文件，并在下一步手动提供名称或通过脚本解析输出。

1.  使用前一步骤中获得的值，现在可以在副本的 **mysql** 客户端中发出适当的命令。在 NDB 8.0.23 及更高版本中，请使用以下 `CHANGE REPLICATION SOURCE TO` 语句：

    ```sql
    mysql*R*> CHANGE REPLICATION SOURCE TO
     ->     SOURCE_LOG_FILE='@file',
     ->     SOURCE_LOG_POS=@pos;
    ```

    在 NDB 8.0.23 之前，您必须使用此处显示的 `CHANGE MASTER TO` 语句：

    ```sql
    mysql*R*> CHANGE MASTER TO
     ->     MASTER_LOG_FILE='@file',
     ->     MASTER_LOG_POS=@pos;
    ```

1.  现在，副本知道从源的哪个二进制日志文件的哪个点开始读取数据，您可以使用此语句使副本开始复制：

    ```sql
    mysql*R*> START SLAVE;
    ```

    从 NDB 8.0.22 开始，您还可以使用以下语句：

    ```sql
    mysql*R*> START REPLICA;
    ```

要在第二个复制通道上执行备份和恢复，只需重复这些步骤，根据需要用次要源和副本的主机名和 ID 替换主要源和副本服务器的名称，并在它们上运行前面的语句。

有关执行集群备份和从备份中恢复集群的更多信息，请参阅第 25.6.8 节，“NDB 集群的在线备份”。
