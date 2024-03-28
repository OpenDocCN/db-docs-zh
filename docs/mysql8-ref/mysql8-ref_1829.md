# 25.7.5 准备 NDB 集群进行复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-preparation.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-preparation.html)

准备 NDB 集群进行复制包括以下步骤：

1.  检查所有 MySQL 服务器的版本兼容性（参见第 25.7.2 节，“NDB 集群复制的一般要求”）。

1.  在源集群上创建一个具有适当权限的复制帐户，使用以下两个 SQL 语句：

    ```sql
    mysql*S*> CREATE USER '*replica_user*'@'*replica_host*'
     -> IDENTIFIED BY '*replica_password*';

    mysql*S*> GRANT REPLICATION SLAVE ON *.*
     -> TO '*replica_user*'@'*replica_host*';
    ```

    在上一条语句中，*`replica_user`*是复制帐户用户名，*`replica_host`*是复制的主机名或 IP 地址，*`replica_password`*是要分配给此帐户的密码。

    例如，要创建一个名为`myreplica`的复制用户帐户，从名为`replica-host`的主机登录，并使用密码`53cr37`，请使用以下`CREATE USER`和`GRANT`语句：

    ```sql
    mysql*S*> CREATE USER 'myreplica'@'replica-host'
     -> IDENTIFIED BY '53cr37';

    mysql*S*> GRANT REPLICATION SLAVE ON *.*
     -> TO 'myreplica'@'replica-host';
    ```

    出于安全原因，最好使用一个唯一的用户帐户—不用于任何其他目的—用于复制帐户。

1.  设置复制使用源。使用**mysql**客户端，可以通过`CHANGE REPLICATION SOURCE TO`语句（从 NDB 8.0.23 开始）或`CHANGE MASTER TO`语句（在 NDB 8.0.23 之前）来实现：

    ```sql
    mysql*R*> CHANGE MASTER TO
     -> MASTER_HOST='*source_host*',
     -> MASTER_PORT=*source_port*,
     -> MASTER_USER='*replica_user*',
     -> MASTER_PASSWORD='*replica_password*';
    ```

    从 NDB 8.0.23 开始，您还可以使用以下语句：

    ```sql
    mysql*R*> CHANGE REPLICATION SOURCE TO
     -> SOURCE_HOST='*source_host*',
     -> SOURCE_PORT=*source_port*,
     -> SOURCE_USER='*replica_user*',
     -> SOURCE_PASSWORD='*replica_password*';
    ```

    在上一条语句中，*`source_host`*是复制源的主机名或 IP 地址，*`source_port`*是复制连接到源时复制使用的端口，*`replica_user`*是在源上为复制设置的用户名，*`replica_password`*是在上一步中为该用户帐户设置的密码。

    例如，要告诉复制使用在上一步中创建的具有复制帐户的 MySQL 服务器，其主机名为`rep-source`，请使用以下语句：

    ```sql
    mysql*R*> CHANGE MASTER TO
     -> MASTER_HOST='rep-source',
     -> MASTER_PORT=3306,
     -> MASTER_USER='myreplica',
     -> MASTER_PASSWORD='53cr37';
    ```

    从 NDB 8.0.23 开始，您还可以使用以下语句：

    ```sql
    mysql*R*> CHANGE REPLICATION SOURCE TO
     -> SOURCE_HOST='rep-source',
     -> SOURCE_PORT=3306,
     -> SOURCE_USER='myreplica',
     -> SOURCE_PASSWORD='53cr37';
    ```

    要查看可与此语句一起使用的选项的完整列表，请参见第 15.4.2.1 节，“CHANGE MASTER TO 语句”。

    为了提供复制备份功能，还需要在开始复制过程之前向复制的`my.cnf`文件中添加一个`--ndb-connectstring`选项。有关详细信息，请参见第 25.7.9 节，“NDB 集群复制的 NDB 集群备份”。

    有关副本可以在`my.cnf`中设置的其他选项，请参见第 19.1.6 节，“复制和二进制日志选项和变量”。

1.  如果源集群已在使用中，则可以创建源的备份并将其加载到副本中，以减少副本与源同步所需的时间。如果副本也在运行 NDB 集群，则可以使用第 25.7.9 节，“NDB 集群复制的 NDB 集群备份”中描述的备份和恢复过程来实现这一点。

    ```sql
    ndb-connectstring=*management_host*[:*port*]
    ```

    如果在副本上*不*使用 NDB 集群，则可以使用以下命令在源上创建备份：

    ```sql
    shell*S*> mysqldump --master-data=1
    ```

    然后通过将转储文件复制到副本上导入生成的数据转储。之后，您可以使用**mysql**客户端将数据从转储文件导入到副本数据库中，如下所示，其中*`dump_file`*是使用**mysqldump**在源上生成的文件的名称，*`db_name`*是要复制的数据库的名称：

    ```sql
    shell*R*> mysql -u root -p *db_name* < *dump_file*
    ```

    要查看与**mysqldump**一起使用的完整选项列表，请参见第 6.5.4 节，“mysqldump — A Database Backup Program”。

    注意

    如果您以这种方式将数据复制到副本中，请确保在所有数据加载完成之前停止副本尝试连接到源以开始复制。您可以通过在命令行上使用`--skip-slave-start`选项启动副本，通过在副本的`my.cnf`文件中包含`skip-slave-start`，或者从 NDB 8.0.24 开始，通过设置`skip_slave_start`系统变量来实现这一点。从 NDB 8.0.26 开始，使用`--skip-replica-start`或`skip_replica_start`。一旦数据加载完成，请按照下面两节中概述的额外步骤进行操作。

1.  确保每个充当复制源的 MySQL 服务器都分配了唯一的服务器 ID，并启用了二进制日志记录，使用基于行的格式。（参见 Section 19.2.1, “Replication Formats”.）此外，我们强烈建议启用 `replica_allow_batching` 系统变量（NDB 8.0.26 及更高版本；在 NDB 8.0.26 之前，请使用 `slave_allow_batching`）。从 NDB 8.0.30 开始，默认情况下已启用此功能。

    如果您使用的是 NDB Cluster 8.0.30 之前的版本，您还应考虑增加与 `--ndb-batch-size` 和 `--ndb-blob-write-batch-bytes` 选项一起使用的值。在 NDB 8.0.30 及更高版本中，请使用 `--ndb-replica-batch-size` 来设置副本上用于写入的批量大小，而不是 `--ndb-batch-size`，以及使用 `--ndb-replica-blob-write-batch-bytes` 而不是 `--ndb-blob-write-batch-bytes` 来确定复制应用程序用于写入 blob 数据的批量大小。所有这些选项都可以在源服务器的 `my.cnf` 文件中设置，或者在启动源 **mysqld** 进程时通过命令行设置。有关更多信息，请参见 Section 25.7.6, “Starting NDB Cluster Replication (Single Replication Channel)”")。
