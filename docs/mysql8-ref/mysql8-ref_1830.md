# 25.7.6 启动 NDB 集群复制（单个复制通道）

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-starting.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-starting.html)

本节概述了使用单个复制通道启动 NDB 集群复制的过程。

1.  通过发出以下命令启动 MySQL 复制源服务器，其中*`id`*是该服务器的唯一 ID（参见第 25.7.2 节，“NDB 集群复制的一般要求”）：

    ```sql
    shell*S*> mysqld --ndbcluster --server-id=*id* \
            --log-bin --ndb-log-bin &
    ```

    这将使用正确的日志格式启动服务器的**mysqld**进程，并启用二进制日志记录。在 NDB 8.0 中，还需要显式启用对`NDB`表更新的日志记录，使用`--ndb-log-bin`选项；这是与 NDB 集群以前版本的一个变化，以前版本中此选项默认启用。

    注意

    你也可以使用`--binlog-format=MIXED`来启动源，这样在集群之间复制时会自动使用基于行的复制。不支持基于语句的二进制日志记录用于 NDB 集群复制（参见第 25.7.2 节，“NDB 集群复制的一般要求”）。

1.  按照以下方式启动 MySQL 复制服务器：

    ```sql
    shell*R*> mysqld --ndbcluster --server-id=*id* &
    ```

    在刚刚显示的命令中，*`id`*是复制服务器的唯一 ID。在复制上不需要启用日志记录。

    注意

    除非你希望立即开始复制，延迟复制线程的启动直到适当的`START REPLICA`语句已经发出，如下面的第 4 步所述。你可以通过在命令行上使用`--skip-slave-start`选项，通过在复制的`my.cnf`文件中包含`skip-slave-start`，或在 NDB 8.0.24 及更高版本中，通过设置`skip_slave_start`系统变量来实现这一点。在 NDB 8.0.26 及更高版本中，使用`--skip-replica-start`和`skip_replica_start`。

1.  必须将复制服务器与源服务器的复制二进制日志同步。如果源上之前没有运行二进制日志记录，请在复制服务器上运行以下语句：

    ```sql
    mysql*R*> CHANGE MASTER TO
     -> MASTER_LOG_FILE='',
     -> MASTER_LOG_POS=4;
    ```

    从 NDB 8.0.23 开始，你还可以使用以下语句：

    ```sql
    mysql*R*> CHANGE REPLICATION SOURCE TO
     -> SOURCE_LOG_FILE='',
     -> SOURCE_LOG_POS=4;
    ```

    这指示副本从日志的起始点开始读取源服务器的二进制日志。否则，即如果您正在使用备份从源加载数据，请参阅 Section 25.7.8，“NDB Cluster 复制实现故障切换”，了解在这种情况下如何获取`SOURCE_LOG_FILE` | `MASTER_LOG_FILE`和`SOURCE_LOG_POS` | `MASTER_LOG_POS`的正确值。

1.  最后，通过在副本上的**mysql**客户端发出以下命令，指示副本开始应用复制： 

    ```sql
    mysql*R*> START SLAVE;
    ```

    在 NDB 8.0.22 及更高版本中，您还可以使用以下语句：

    ```sql
    mysql*R*> START REPLICA;
    ```

    这也启动了从源到副本的数据和更改的传输。

也可以使用两个复制通道，类似于下一节描述的过程；这种方法与使用单个复制通道的区别在 Section 25.7.7，“NDB Cluster 复制使用两个复制通道”中有所涵盖。

也可以通过启用批量更新来提高集群复制性能。可以通过在副本的**mysqld**进程上设置系统变量`replica_allow_batching`（NDB 8.0.26 及更高版本）或`slave_allow_batching`（NDB 8.0.26 之前）来实现。通常，更新在接收到时立即应用。但是，使用批处理会导致每个批次应用 32 KB 的更新；这可能会导致更高的吞吐量和更少的 CPU 使用，特别是在单个更新相对较小的情况下。

注意

批处理是基于每个时代的基础进行的；属于多个事务的更新可以作为同一批发送。

当达到时代结束时，所有未完成的更新都会被应用，即使更新总量不到 32 KB。

批处理可以在运行时打开和关闭。要在运行时激活它，您可以使用以下两个语句之一：

```sql
SET GLOBAL slave_allow_batching = 1;
SET GLOBAL slave_allow_batching = ON;
```

从 NDB 8.0.26 开始，您可以（并且应该）改用以下语句之一：

```sql
SET GLOBAL replica_allow_batching = 1;
SET GLOBAL replica_allow_batching = ON;
```

如果特定批次导致问题（例如，效果似乎没有正确复制的语句），可以使用以下两个语句之一来停用批处理：

```sql
SET GLOBAL slave_allow_batching = 0;
SET GLOBAL slave_allow_batching = OFF;
```

从 NDB 8.0.26 开始，您可以（并且应该）改用以下语句之一：

```sql
SET GLOBAL replica_allow_batching = 0;
SET GLOBAL replica_allow_batching = OFF;
```

您可以通过适当的`SHOW VARIABLES`语句来检查当前是否正在使用批处理，例如：

```sql
mysql> SHOW VARIABLES LIKE 'slave%';
```

在 NDB 8.0.26 及更高版本中，请使用以下语句：

```sql
mysql> SHOW VARIABLES LIKE 'replica%';
```
