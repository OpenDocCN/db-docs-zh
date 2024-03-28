> [`dev.mysql.com/doc/refman/8.0/en/replication-asynchronous-connection-failover-source.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-asynchronous-connection-failover-source.html)

#### 19.4.9.1 异步连接源的故障转移

要激活复制通道的异步连接故障转移，在`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（MySQL 8.0.23 之前）中设置`SOURCE_CONNECTION_AUTO_FAILOVER=1`。通道必须使用 GTID 自动定位（`SOURCE_AUTO_POSITION = 1` | `MASTER_AUTO_POSITION = 1`）。

重要提示

当与源的现有连接失败时，复制品首先重试由`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句的`SOURCE_RETRY_COUNT` | `MASTER_RETRY_COUNT`选项指定的次数。尝试之间的间隔由`SOURCE_CONNECT_RETRY` | `MASTER_CONNECT_RETRY`选项设置。当这些尝试耗尽时，异步连接故障转移机制接管。请注意，这些选项的默认值是为了连接到单个源而设计的，使复制品在 60 天内重试相同的连接。为了确保可以及时激活异步连接故障转移机制，请将`SOURCE_RETRY_COUNT` | `MASTER_RETRY_COUNT`和`SOURCE_CONNECT_RETRY` | `MASTER_CONNECT_RETRY`设置为最小值，只允许与同一源进行少量重试尝试，以防连接故障是由瞬时网络中断引起的。适当的值是`SOURCE_RETRY_COUNT=3` | `MASTER_RETRY_COUNT=3`和`SOURCE_CONNECT_RETRY=10` | `MASTER_CONNECT_RETRY=10`，使复制品在 10 秒间隔内重试 3 次连接。

您还需要为复制通道设置源列表，以指定可用于故障转移的源。您可以使用`asynchronous_connection_failover_add_source`和`asynchronous_connection_failover_delete_source`函数来设置和管理源列表，以添加和删除单个复制源服务器。要添加和删除受管理的服务器组，请改用`asynchronous_connection_failover_add_managed`和`asynchronous_connection_failover_delete_managed`函数。

函数命名相关复制通道，并指定要添加到或从通道源列表中删除的 MySQL 实例的主机名、端口号、网络命名空间和加权优先级（1-100，100 为最高优先级）。对于受管组，您还需要指定受管服务的类型（目前仅有 Group Replication 可用），以及受管组的标识符（对于 Group Replication，这是`group_replication_group_name`系统变量的值）。当您添加受管组时，只需添加一个组成员，副本会自动添加当前组成员的其余部分。当您删除受管组时，将一起删除整个组。

在 MySQL 8.0.22 中，异步连接故障转移机制在副本与源的连接失败后被激活，并发出`START REPLICA`语句尝试连接到新源。在此版本中，如果由于源停止或由于网络故障导致复制接收线程停止，连接将转移。在其他情况下，例如当复制线程被`STOP REPLICA`语句停止时，连接不会转移。

从 MySQL 8.0.23 开始，异步连接故障转移机制还会在源列表中另一个可用服务器具有更高优先级（权重）设置时转移连接。此功能确保副本始终连接到最合适的源服务器，并适用于受管组和单个（非受管）服务器。对于受管组，源的权重根据其是主服务器还是从服务器而分配。因此，假设您设置受管组以给予主服务器更高的权重，从服务器更低的权重，当主服务器更改时，新主服务器被分配更高的权重，因此副本将连接到新主服务器。如果当前连接的受管源服务器离开受管组，或者不再是受管组中的大多数，异步连接故障转移机制还会更改连接。

在连接故障时，通道的源列表中列出的备用源中具有最高优先级（权重）设置的源将被选择用于第一次连接尝试。 复制品首先检查它是否可以连接到源服务器，或者在管理组的情况下，源服务器在组中是否具有`ONLINE`状态（而不是`RECOVERING`或不可用）。 如果最高加权源不可用，则复制品将按权重降序尝试所有列出的源，然后再从最高加权源开始。 如果多个源具有相同的权重，则复制品会随机排序它们。 如果复制品需要重新开始遍历列表，则它会包括并重试原始连接故障发生的源。

源列表存储在`mysql.replication_asynchronous_connection_failover`和`mysql.replication_asynchronous_connection_failover_managed`表中，并可以在性能模式`replication_asynchronous_connection_failover`和`replication_asynchronous_connection_failover_managed`表中查看。 复制品使用监视线程跟踪管理组的成员资格并更新源列表（`thread/sql/replica_monitor`）。 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句的`SOURCE_CONNECTION_AUTO_FAILOVER`选项设置和源列表在远程克隆操作期间传输到复制品的克隆中。
