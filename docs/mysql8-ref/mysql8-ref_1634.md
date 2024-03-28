> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-exclusive-to-cluster.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-exclusive-to-cluster.html)

#### 25.2.7.8 NDB Cluster 专有问题

以下是特定于`NDB`存储引擎的限制：

+   **机器架构。** 集群中使用的所有机器必须具有相同的架构。也就是说，托管节点的所有机器必须是大端或小端，不能混合使用。例如，不能在运行在 PowerPC 上的管理节点上指导在 x86 机器上运行的数据节点。这个限制不适用于简单运行**mysql**或其他可能访问集群 SQL 节点的客户端的机器。

+   **二进制日志。** NDB Cluster 在二进制日志方面具有以下限制或限制：

    +   NDB Cluster 无法为具有`BLOB`列但没有主键的表生成二进制日志。

    +   只有以下模式操作会记录在集群二进制日志中，该日志*不*在执行语句的**mysqld**上：

        +   `CREATE TABLE`

        +   `ALTER TABLE`

        +   `DROP TABLE`

        +   `CREATE DATABASE` / `CREATE SCHEMA`

        +   `DROP DATABASE` / `DROP SCHEMA`

        +   `CREATE TABLESPACE`

        +   `ALTER TABLESPACE`

        +   `DROP TABLESPACE`

        +   `CREATE LOGFILE GROUP`

        +   `ALTER LOGFILE GROUP`

        +   `DROP LOGFILE GROUP`

+   **模式操作。** 在任何数据节点重新启动时，模式操作（DDL 语句）将被拒绝。在执行在线升级或降级时也不支持模式操作。

+   **分片副本数量。** 由`NoOfReplicas`数据节点配置参数确定的分片副本数量是 NDB 集群存储的所有数据的副本数量。将此参数设置为 1 表示只有一个副本；在这种情况下，不提供冗余，且数据节点丢失会导致数据丢失。为了保证冗余性，即使数据节点失败也能保留数据，将此参数设置为 2，这是生产环境中的默认和推荐值。

    将`NoOfReplicas`设置为大于 2 的值是支持的（最多为 4），但不必要以防止数据丢失。

请参阅第 25.2.7.10 节，“关于多个 NDB 集群节点的限制”。
