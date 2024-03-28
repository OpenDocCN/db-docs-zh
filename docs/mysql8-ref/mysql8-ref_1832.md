# 25.7.8 使用 NDB 集群复制实现故障转移

> 译文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-failover.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-failover.html)

如果主要集群复制过程失败，可以切换到次要复制通道。以下过程描述了完成此操作所需的步骤。

1.  获取最近全局检查点（GCP）的时间。也就是说，您需要确定副本集群上的`ndb_apply_status`表中的最新时代，可以使用以下查询找到：

    ```sql
    mysql*R'*> SELECT @latest:=MAX(epoch)
     ->        FROM mysql.ndb_apply_status;
    ```

    在循环复制拓扑中，每个主机上都运行源和副本时，当您使用`ndb_log_apply_status=1`时，NDB 集群时代会写入副本的二进制日志中。这意味着`ndb_apply_status`表包含了此主机上的副本以及任何其他充当此主机上运行的复制源服务器的副本的主机的信息。

    在这种情况下，您需要确定此副本上的最新时代，排除在此副本的二进制日志中未列在`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句的`IGNORE_SERVER_IDS`选项中的任何其他副本的时代。排除这些时代的原因是，`mysql.ndb_apply_status`表中具有与`IGNORE_SERVER_IDS`列表中的匹配的服务器 ID 的行，从`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句中的本副本的源准备的行也被视为来自本地服务器，除了具有副本自身服务器 ID 的行。您可以从`SHOW REPLICA STATUS`的输出中检索此列表作为`Replicate_Ignore_Server_Ids`。我们假设您已经获得了此列表，并将其替换为此处显示的查询中的*`ignore_server_ids`*，该查询与上一个查询的版本一样，将最大时代选择到名为`@latest`的变量中：

    ```sql
    mysql*R'*> SELECT @latest:=MAX(epoch)
     ->        FROM mysql.ndb_apply_status
     ->        WHERE server_id NOT IN (*ignore_server_ids*);
    ```

    在某些情况下，使用要包含的服务器 ID 列表和在前述查询的`WHERE`条件中的`server_id IN *`server_id_list`*`可能更简单或更有效（或两者兼有）。

1.  使用第 1 步中显示的查询获取的信息，从源集群的`ndb_binlog_index`表中获取相应的记录。

    您可以使用以下查询从源集群的`ndb_binlog_index`表中获取所需的记录：

    ```sql
    mysql*S'*> SELECT
     ->     @file:=SUBSTRING_INDEX(next_file, '/', -1),
     ->     @pos:=next_position
     -> FROM mysql.ndb_binlog_index
     -> WHERE epoch = @latest;
    ```

    这些是自主要复制通道故障以来在源上保存的记录。我们在这里使用了一个用户变量 `@latest` 来表示第 1 步中获得的值。当然，一个 **mysqld** 实例无法直接访问在另一个服务器实例上设置的用户变量。这些值必须手动或通过应用程序“插入”到第二个查询中。

    重要

    在执行`START REPLICA`之前，您必须确保副本**mysqld**使用`--slave-skip-errors=ddl_exist_errors`启动。否则，复制可能因重复的 DDL 错误而停止。

1.  现在可以通过在次要副本服务器上运行以下查询来同步次要通道：

    ```sql
    mysql*R'*> CHANGE MASTER TO
          ->     MASTER_LOG_FILE='@file',
     ->     MASTER_LOG_POS=@pos;
    ```

    在 NDB 8.0.23 及更高版本中，您还可以使用此处显示的语句：

    ```sql
    mysql*R'*> CHANGE REPLICATION SOURCE TO
          ->     SOURCE_LOG_FILE='@file',
     ->     SOURCE_LOG_POS=@pos;
    ```

    再次我们使用了用户变量（在这种情况下是 `@file` 和 `@pos`）来表示第 2 步中获得的值，并在第 3 步中应用；在实践中，这些值必须手动插入或使用能够访问涉及的两个服务器的应用程序。

    注意

    `@file` 是一个字符串值，例如 `'/var/log/mysql/replication-source-bin.00001'`，因此在 SQL 或应用程序代码中使用时必须加引号。然而，由 `@pos` 表示的值则不应加引号。尽管 MySQL 通常会尝试将字符串转换为数字，但这种情况是一个例外。

1.  您现在可以通过在次要副本上发出适当的命令来启动次要通道的复制**mysqld**：

    ```sql
    mysql*R'*> START SLAVE;
    ```

    在 NDB 8.0.22 或更高版本中，您还可以使用以下语句：

    ```sql
    mysql*R'*> START REPLICA;
    ```

一旦次要复制通道激活，您可以调查主要通道的故障并进行修复。执行这些精确的操作取决于主要通道失败的原因。

警告

只有在主要复制通道失败时才应启动次要复制通道。同时运行多个复制通道可能导致副本上创建不需要的重复记录。

如果故障仅限于单个服务器，则理论上应该可以从 *`S`* 复制到 *`R'`*，或者从 *`S'`* 复制到 *`R`*。
