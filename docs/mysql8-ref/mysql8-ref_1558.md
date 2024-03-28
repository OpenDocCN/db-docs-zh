> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-upgrading-member.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-upgrading-member.html)

#### 20.8.3.2 升级组复制成员

本节解释了升级组成员所需的步骤。此过程是第 20.8.3.3 节“组复制在线升级方法”描述的方法的一部分。升级组成员的过程对所有方法都是通用的，并首先进行解释。加入升级成员的方式可能取决于您正在遵循的方法，以及诸如组是在单主还是多主模式下运行等其他因素。您如何升级服务器实例，无论是使用就地升级还是预置方法，都不会影响此处描述的方法。

升级成员的过程包括将其从组中移除，按照您选择的升级成员的方法进行升级，然后将升级后的成员重新加入组。在单主组中升级成员的推荐顺序是先升级所有次要成员，然后最后再升级主要成员。如果在次要成员之前升级主要成员，则会选择使用旧版 MySQL 版本的新主要成员，但这一步是不必要的。

要升级组的成员：

+   连接客户端到组成员并发出`STOP GROUP_REPLICATION`。在继续之前，请通过监视`replication_group_members`表确保成员的状态为`OFFLINE`。

+   禁用自动启动组复制，以便您可以安全地在升级后连接到成员并进行配置，而不会在设置`group_replication_start_on_boot=0`后重新加入组。

    重要

    如果升级的成员设置了`group_replication_start_on_boot=1`，那么在你执行 MySQL 升级过程之前，它可能会重新加入组，并可能导致问题。例如，如果升级失败并且服务器再次重启，那么可能会尝试加入组的是一个可能损坏的服务器。

+   停止成员，例如使用**mysqladmin shutdown**或`SHUTDOWN`语句。组中的其他成员继续运行。

+   使用就地或配置方法升级成员。有关详细信息，请参见第三章，*升级 MySQL*。在重新启动升级后的成员时，因为`group_replication_start_on_boot`设置为 0，Group Replication 不会在实例上启动，因此它不会重新加入组。

+   一旦在成员上执行了 MySQL 升级过程，必须将`group_replication_start_on_boot`设置为 1，以确保在重新启动后正确启动 Group Replication。重新启动成员。

+   连接到升级后的成员并发出`START GROUP_REPLICATION`。这将重新加入成员到组中。升级服务器上已经存在 Group Replication 元数据，因此通常不需要重新配置 Group Replication。服务器必须赶上在其离线时组处理的任何事务。一旦赶上组，它就成为组的在线成员。

    注意

    升级服务器所需的时间越长，该成员离线的时间就越长，因此在重新添加到组中时，服务器需要花费更多时间来赶上。

当升级后的成员加入任何运行较早 MySQL 服务器版本的组时，升级后的成员会以`super_read_only=on`加入。这确保在所有成员都运行新版本之前，不会向升级后的成员进行写入。在多主模式组中，当升级成功完成并且组准备好处理事务时，打算作为可写主节点的成员必须设置为读写模式。从 MySQL 8.0.17 开始，当组的所有成员都升级到相同版本时，它们都会自动切换回读写模式。对于早期版本，您必须手动将每个成员设置为读写模式。连接到每个成员并发出：

```sql
SET GLOBAL super_read_only=OFF;
```
