# 20.5.2 重新启动组

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-restarting-group.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-restarting-group.html)

Group Replication 旨在确保数据库服务持续可用，即使组成该组的某些服务器由于计划维护或意外问题而无法参与其中。只要剩余成员占据组的大多数，他们就可以选举新的主服务器并继续作为一个组运行。然而，如果复制组的每个成员都离开组，并且每个成员都通过`STOP GROUP_REPLICATION`语句或系统关闭停止了 Group Replication，那么该组现在只存在于理论上，作为成员上的一个配置。在这种情况下，要重新创建该组，必须像第一次启动一样通过引导启动。

首次引导组与第二次或后续引导组之间的区别在于，后者情况下，关闭的组的成员可能具有不同的事务集，取决于它们停止或失败的顺序。如果成员具有其他组成员上不存在的事务，则无法加入组。对于 Group Replication，这包括已提交和应用的事务，这些事务在`gtid_executed` GTID 集中，以及已经认证但尚未应用的事务，这些事务在`group_replication_applier`通道中。事务何时提交取决于为组设置的事务一致性级别（参见第 20.5.3 节，“事务一致性保证”）。然而，Group Replication 组成员永远不会删除已经认证的事务，这是成员承诺提交事务的声明。

因此，必须从最新的成员开始重新启动复制组，即执行最多事务并获得认证的成员。然后，事务较少的成员可以通过分布式恢复加入并赶上他们缺少的事务。不能假设组的最后已知主成员是组中最新的成员，因为比主成员关闭时间更晚的成员可能有更多的事务。因此，必须重新启动每个成员以检查事务，比较所有事务集，并确定最新的成员。然后可以使用该成员来引导组。

在每个成员关闭后，按照以下步骤安全地重新启动复制组。

1.  依次对每个组成员执行以下步骤，顺序不限：

    1.  连接客户端到组成员。如果 Group Replication 尚未停止，请执行 `STOP GROUP_REPLICATION` 语句并等待 Group Replication 停止。

    1.  编辑 MySQL 服务器配置文件（通常在 Linux 和 Unix 系统上命名为 `my.cnf`，在 Windows 系统上命名为 `my.ini`），并设置系统变量 `group_replication_start_on_boot=OFF`。此设置防止 MySQL 服务器启动时启动 Group Replication，这是默认设置。

        如果无法在系统上更改该设置，则可以允许服务器尝试启动 Group Replication，这将失败，因为组已完全关闭且尚未引导。如果采用这种方法，请勿在此阶段在任何服务器上设置 `group_replication_bootstrap_group=ON`。

    1.  启动 MySQL 服务器实例，并验证 Group Replication 尚未启动（或启动失败）。此阶段不要启动 Group Replication。

    1.  从组成员收集以下信息：

        +   `gtid_executed` GTID 集的内容。您可以通过执行以下语句获取：

            ```sql
            mysql> SELECT @@GLOBAL.GTID_EXECUTED
            ```

        +   `group_replication_applier` 通道上的已认证事务集。您可以通过执行以下语句获取：

            ```sql
            mysql> SELECT received_transaction_set FROM \
                    performance_schema.replication_connection_status WHERE \
                    channel_name="group_replication_applier";
            ```

1.  当您从所有组成员收集了事务集后，比较它们以找出哪个成员具有最大的事务集，包括已执行的事务（`gtid_executed`）和已认证的事务（在 `group_replication_applier` 通道上）。您可以通过查看 GTID 手动执行此操作，或者使用存储函数比较 GTID 集，如 Section 19.1.3.8, “Stored Function Examples to Manipulate GTIDs” 中所述。

1.  使用具有最大事务集的成员引导组，通过连接客户端到组成员并执行以下语句：

    ```sql
    mysql> SET GLOBAL group_replication_bootstrap_group=ON;
    mysql> START GROUP_REPLICATION;
    mysql> SET GLOBAL group_replication_bootstrap_group=OFF;
    ```

    非常重要的是不要将设置 `group_replication_bootstrap_group=ON` 存储在配置文件中，否则当服务器再次重启时���将设置一个具有相同名称的第二个组。

1.  要验证该组现在存在并包含此创始成员，请在引导它的成员上执行此语句：

    ```sql
    mysql> SELECT * FROM performance_schema.replication_group_members;
    ```

1.  通过在每个成员上执行 `START GROUP_REPLICATION` 语句，以任意顺序将其他每个成员重新添加到组中：

    ```sql
    mysql> START GROUP_REPLICATION;
    ```

1.  要验证每个成员是否已加入组，请在任何成员上执行此语句：

    ```sql
    mysql> SELECT * FROM performance_schema.replication_group_members;
    ```

1.  当成员重新加入组后，如果您编辑了他们的配置文件以设置`group_replication_start_on_boot=OFF`，您可以再次编辑它们以设置`ON`（或者移除该系统变量，因为`ON`是默认值）。
