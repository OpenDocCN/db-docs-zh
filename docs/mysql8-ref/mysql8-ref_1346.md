> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-gtids-auto-positioning.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-gtids-auto-positioning.html)

#### 19.1.3.3 GTID 自动定位

GTID 取代了以前需要确定源和副本之间数据流开始、停止或恢复点的文件偏移对。当使用 GTID 时，副本与源同步所需的所有信息都直接从复制数据流中获取。

要使用基于 GTID 的复制启动副本，需要在 `CHANGE REPLICATION SOURCE TO` 语句（从 MySQL 8.0.23 开始）或 `CHANGE MASTER TO` 语句（MySQL 8.0.23 之前）中启用 `SOURCE_AUTO_POSITION` | `MASTER_AUTO_POSITION` 选项。另外的 `SOURCE_LOG_FILE` | `MASTER_LOG_FILE` 和 `SOURCE_LOG_POS` | `MASTER_LOG_POS` 选项指定了日志文件的名称和文件内的起始位置，但是使用 GTID 时，副本不需要这些非本地数据。有关使用基于 GTID 的复制配置和启动源和副本的完整说明，请参见 Section 19.1.3.4, “Setting Up Replication Using GTIDs”。

`SOURCE_AUTO_POSITION` | `MASTER_AUTO_POSITION` 选项默认情况下是禁用的。如果在副本上启用了多源复制，则需要为每个适用的复制通道设置该选项。再次禁用 `SOURCE_AUTO_POSITION` | `MASTER_AUTO_POSITION` 选项会导致副本恢复到基于位置的复制；这意味着当 `GTID_ONLY=ON` 时，某些位置可能被标记为无效，在这种情况下，当禁用 `SOURCE_AUTO_POSITION` | `MASTER_AUTO_POSITION` 时，还必须同时指定 `SOURCE_LOG_FILE` | `MASTER_LOG_FILE` 和 `SOURCE_LOG_POS` | `MASTER_LOG_POS`。

当副本启用了 GTIDs（`GTID_MODE=ON`，`ON_PERMISSIVE`或`OFF_PERMISSIVE`）并启用了`MASTER_AUTO_POSITION`选项时，自动定位将被激活以连接到源端。源端必须设置`GTID_MODE=ON`才能成功连接。在初始握手中，副本发送一个包含其已经接收、提交或两者都有的交易的 GTID 集合。这个 GTID 集合等于`gtid_executed`系统变量（`@@GLOBAL.gtid_executed`）中的 GTID 集合和性能模式中记录的 GTID 集合的并集，性能模式中的 GTID 集合是作为已接收交易记录的（执行`SELECT RECEIVED_TRANSACTION_SET FROM PERFORMANCE_SCHEMA.replication_connection_status`语句的结果）。

源端通过发送其二进制日志中记录的所有交易来响应，这些交易的 GTID 不包含在副本发送的 GTID 集合中。为此，源端首先通过检查其二进制日志文件的头部中的`Previous_gtids_log_event`来确定要开始处理的适当二进制日志文件，从最近的文件开始检查。当源端找到第一个不包含副本缺失交易的`Previous_gtids_log_event`时，它就从那个二进制日志文件开始。这种方法是高效的，只有在副本落后源端很多个二进制日志文件时才会花费大量时间。然后，源端读取该二进制日志文件和后续文件中的交易，直到当前文件，发送副本缺失的带有 GTID 的交易，并跳过副本发送的 GTID 集合中的交易。副本接收到第一个缺失交易的经过时间取决于其在二进制日志文件中的偏移量。这种交换确保源端只发送副本尚未接收或提交的带有 GTID 的交易。如果副本从多个源接收交易，如钻石拓扑结构的情况下，自动跳过功能确保交易不会被应用两次。

如果应该由源发送的任何事务已经从源的二进制日志中清除，或者通过其他方法添加到`gtid_purged`系统变量的 GTID 集合中，源会向副本发送错误`ER_MASTER_HAS_PURGED_REQUIRED_GTIDS`，并且复制不会启动。缺失的已清除事务的 GTID 将在源的错误日志中被识别并列在警告消息`ER_FOUND_MISSING_GTIDS`中。副本无法自动从此错误中恢复，因为需要追赶源的事务历史的部分已被清除。尝试重新连接而没有启用`MASTER_AUTO_POSITION`选项只会导致副本上的已清除事务的丢失。从这种情况中恢复的正确方法是让副本从另一个源复制`ER_FOUND_MISSING_GTIDS`消息中列出的缺失事务，或者让副本被一个从更近期备份创建的新副本所取代。考虑修改源上的二进制日志过期时间（`binlog_expire_logs_seconds`）以确保不再发生这种情况。

如果在交换事务过程中发现复制品已经收到或提交了具有源 GTID 中 UUID 的事务，但源本身没有记录它们，源会向复制品发送错误`ER_SLAVE_HAS_MORE_GTIDS_THAN_MASTER`，复制不会开始。如果源没有设置`sync_binlog=1`并遇到断电或操作系统崩溃，且丢失了尚未同步到二进制日志文件的已提交事务，但已被复制品接收，那么就会出现这种情况。如果在源重新启动后有任何客户端提交事务，可能导致源和复制品使用相同的 GTID 进行不同的事务，源和复制品可能会发散。从这种情况中恢复的正确方法是手动检查源和复制品是否发散。如果现在对不同事务使用相同的 GTID，则需要根据需要对单个事务执行手动冲突解决，或者从复制拓扑中删除源或复制品。如果问题仅在源上缺少事务，则可以将源变为复制品，使其赶上复制拓扑中的其他服务器，然后根据需要再次将其变为源。

对于钻石拓扑结构中的多源复制（其中复制品从两个或更多源复制，这些源又从一个共同源复制），当使用基于 GTID 的复制时，请确保多源复制上的所有通道上的任何复制过滤器或其他通道配置都是相同的。使用基于 GTID 的复制时，过滤器仅应用于事务数据，而 GTID 不会被过滤掉。这是为了使复制品的 GTID 集与源保持一致，这意味着可以使用 GTID 自动定位而无需每次重新获取被过滤的事务。在下游复制品是多源的情况下，并且在钻石拓扑结构中从多个源接收相同事务的情况下，下游复制品现在具有事务的多个版本，结果取决于哪个通道首先应用该事务。尝试的第二个通道通过使用 GTID 自动跳过来跳过该事务，因为该事务的 GTID 已被第一个通道添加到`gtid_executed`集中。在通道上具有相同过滤器的情况下，没有问题，因为所有事务的所有版本都包含相同的数据，因此结果是相同的。然而，在通道上具有不同过滤器的情况下，数据库可能变得不一致，复制可能会挂起。
