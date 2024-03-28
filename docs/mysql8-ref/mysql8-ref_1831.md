# 25.7.7 使用两个复制通道进行 NDB 集群复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-two-channels.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-two-channels.html)

在一个更完整的示例场景中，我们设想使用两个复制通道来提供冗余，从而防范单个复制通道可能的故障。这需要总共四个复制服务器，两个源服务器在源集群上，两个复制品服务器在复制品集群上。在接下来的讨论中，我们假设分配了如下所示的唯一标识符：

**表 25.73 文本中描述的 NDB 集群复制服务器**

| 服务器 ID | 描述 |
| --- | --- |
| 1 | 源 - 主要复制通道（*S*） |
| 2 | 源 - 次要复制通道（*S'*） |
| 3 | 复制品 - 主要复制通道（*R*） |
| 4 | 复制品 - 次要复制通道（*R'*） |

使用两个通道设置复制与设置单个复制通道并没有根本不同。首先，必须启动主和次要复制源服务器的**mysqld**进程，然后启动主和次要复制品的进程。可以通过在每个复制品上发出`START REPLICA`语句来启动复制进程。下面显示了需要发出的命令和顺序：

1.  启动主复制源：

    ```sql
    shell*S*> mysqld --ndbcluster --server-id=1 \
                   --log-bin &
    ```

1.  启动次要复制源：

    ```sql
    shell*S'*> mysqld --ndbcluster --server-id=2 \
                   --log-bin &
    ```

1.  启动主复制品服务器：

    ```sql
    shell*R*> mysqld --ndbcluster --server-id=3 \
                   --skip-slave-start &
    ```

1.  启动次要复制品服务器：

    ```sql
    shell*R'*> mysqld --ndbcluster --server-id=4 \
                    --skip-slave-start &
    ```

1.  最后，通过在主复制品上执行`START REPLICA`语句来启动主通道上的复制。

    ```sql
    mysql*R*> START SLAVE;
    ```

    从 NDB 8.0.22 开始，您还可以使用以下语句：

    ```sql
    mysql*R*> START REPLICA;
    ```

    警告

    此时只需启动主通道。只有在主复制通道失败时才需要启动次要复制通道，如第 25.7.8 节“使用 NDB 集群复制实现故障切换”中所述。同时运行多个复制通道可能导致在复制品上创建不需要的重复记录。

如前所述，在复制品上不需要启用二进制日志记录。
