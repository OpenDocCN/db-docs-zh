# 20.3.2 Group Replication 限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-limitations.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-limitations.html)

+   组大小限制

+   事务大小限制

Group Replication 存在以下已知限制。请注意，在故障转移事件期间，多主模式组的限制和问题也可能适用于单主模式集群，而新选举的主节点会清空其来自旧主节点的应用程序队列。

提示

Group Replication 是基于 GTID 的复制构建的，因此您还应该注意 Section 19.1.3.7, “使用 GTID 进行复制的限制”。

+   **`--upgrade=MINIMAL`选项。** Group Replication 无法在使用 MINIMAL 选项(`--upgrade=MINIMAL`)进行 MySQL Server 升级后启动，因为该选项不会升级复制内部所依赖的系统表。

+   **间隙锁。** Group Replication 的并发事务认证过程不考虑间隙锁，因为间隙锁的信息在`InnoDB`之外不可用。有关更多信息，请参阅间隙锁。

    注意

    对于多主模式下的组，除非您的应用程序依赖于`REPEATABLE READ`语义，我们建议在 Group Replication 中使用`READ COMMITTED`隔离级别。 InnoDB 在`READ COMMITTED`中不使用间隙锁，这使得 InnoDB 内部的本地冲突检测与 Group Replication 执行的分布式冲突检测保持一致。对于单主模式下的组，只有主节点接受写操作，因此`READ COMMITTED`隔离级别对于 Group Replication 并不重要。

+   **表锁和命名锁。** 认证过程不考虑表锁（参见 Section 15.3.6, “LOCK TABLES 和 UNLOCK TABLES 语句”）或命名锁（参见`GET_LOCK()`）。

+   **二进制日志校验和。** 截至 MySQL 8.0.20，Group Replication 无法使用校验和，也不支持二进制日志中的校验和，因此在配置服务器实例成为组成员时，必须设置`binlog_checksum=NONE`。从 MySQL 8.0.21 开始，Group Replication 支持校验和，因此组成员可以使用默认设置`binlog_checksum=CRC32`。`binlog_checksum` 的设置不必对组中的所有成员相同。

    当校验和可用时，Group Replication 不会使用它们来验证`group_replication_applier`通道上的传入事件，因为事件是从多个来源写入到中继日志中的，而在它们实际写入到原始服务器的二进制日志之前，校验和是不会生成的。校验和用于验证`group_replication_recovery`通道上的事件的完整性，以及组成员上的任何其他复制通道上的事件。

+   **可串行化隔离级别。** `SERIALIZABLE` 隔离级别在多主组中默认不受支持。将事务隔离级别设置为`SERIALIZABLE`会配置 Group Replication 拒绝提交事务。

+   **并发 DDL 与 DML 操作。** 在使用多主模式时，不支持针对同一对象执行并发数据定义语句和数据操作语句，但在不同的服务器上执行。在对象上执行数据定义语言（DDL）语句期间，在不同服务器实例上执行相同对象的并发数据操作语言（DML）存在冲突的风险，因为在不同实例上执行的冲突 DDL 可能不会被检测到。

+   **具有级联约束的外键。** 多主模式组（所有成员都配置为`group_replication_single_primary_mode=OFF`）不支持具有多级外键依赖关系的表，特别是定义了`CASCADING`外键约束的表。这是因为由多主模式组执行的导致级联操作的外键约束可能导致未检测到的冲突，并导致组成员之间的数据不一致。因此，我们建议在用于多主模式组的服务器实例上设置`group_replication_enforce_update_everywhere_checks=ON`以避免未检测到的冲突。

    在单主模式下，这不是问题，因为它不允许并发写入到组的多个成员，因此不存在未检测到的冲突风险。

+   **多主模式死锁。** 当一个组以多主模式运行时，`SELECT .. FOR UPDATE`语句可能导致死锁。这是因为锁不在组的成员之间共享，因此对于这样的语句的期望可能无法实现。

+   **复制过滤器。** 不能在配置为 Group Replication 的 MySQL 服务器实例上使用全局复制过滤器，因为在某些服务器上过滤事务会导致组无法达成一致状态。可以在与 Group Replication 无直接关系的复制通道上使用特定通道的复制过滤器，例如，当一个组成员同时充当组外源的副本时。不能在`group_replication_applier`或`group_replication_recovery`通道上使用。

+   **加密连接。** MySQL Server 自 MySQL 8.0.16 起支持 TLSv1.3 协议，前提是 MySQL 使用了 OpenSSL 1.1.1 或更高版本进行编译。在 MySQL 8.0.16 和 MySQL 8.0.17 中，如果服务器支持 TLSv1.3，则该协议在组通信引擎中不受支持，无法被 Group Replication 使用。从 MySQL 8.0.18 开始，Group Replication 支持 TLSv1.3，可以用于组通信连接和分布式恢复连接。

    在 MySQL 8.0.18 中，TLSv1.3 可以用于 Group Replication 的分布式恢复连接，但`group_replication_recovery_tls_version`和`group_replication_recovery_tls_ciphersuites`系统变量不可用。因此，捐赠服务器必须允许使用至少一个默认启用的 TLSv1.3 密码套件，如第 8.3.2 节“加密连接 TLS 协议和密码套件”中列出的那样。从 MySQL 8.0.19 开始，您可以使用选项配置客户端支持任意选择的密码套件，包括仅使用非默认密码套件。

+   **克隆操作。** Group Replication 启动并管理用于分布式恢复的克隆操作，但已设置为支持克隆的组成员也可以参与用户手动启动的克隆操作。在 MySQL 8.0.20 之前的版本中，如果操作涉及正在运行 Group Replication 的组成员，则无法手动启动克隆操作。从 MySQL 8.0.20 开始，只要克隆操作不会删除并替换接收方的数据，就可以执行此操作。因此，如果 Group Replication 正在运行，则启动克隆操作的语句必须包含`DATA DIRECTORY`子句。请参阅 Section 20.5.4.2.4, “Cloning for Other Purposes”。

#### 组大小限制

单个复制组中可以成为成员的 MySQL 服务器的最大数量为 9。如果更多成员尝试加入组，则其请求将被拒绝。这个限制是通过测试和基准测试确定的，作为组在稳定的局域网上可靠运行的安全边界。

#### 事务大小限制

如果单个事务导致消息内容过大，以至于在 5 秒窗口内无法在组成员之间复制消息，那么成员可能会被怀疑失败，然后被驱逐，仅仅因为它们正忙于处理事务。大型事务也可能导致系统由于内存分配问题而变慢。为避免这些问题，请使用以下缓解措施：

+   如果由于大型消息而发生不必要的驱逐，请使用系统变量`group_replication_member_expel_timeout`在怀疑失败的成员被驱逐之前提供额外的时间。在初始的 5 秒检测期之后，您可以允许多达一小时的时间，然后才将怀疑的成员从组中驱逐。从 MySQL 8.0.21 开始，默认情况下额外允许 5 秒。

+   在可能的情况下，在交由 Group Replication 处理之前，尽量限制事务的大小。例如，将与`LOAD DATA`一起使用的文件拆分为较小的块。

+   使用系统变量`group_replication_transaction_size_limit`来指定组接受的最大事务大小。在 MySQL 8.0 中，此系统变量默认为最大事务大小为 150000000 字节（约 143 MB）。超过此大小的事务将被回滚，并且不会发送到 Group Replication 的 Group Communication System (GCS)以分发给组。根据您需要组容忍的最大消息大小调整此变量的值，要牢记处理事务所需的时间与其大小成正比。

+   使用系统变量`group_replication_compression_threshold`来指定应用压缩的消息大小阈值。该系统变量默认为 1000000 字节（1 MB），因此大消息会自动进行压缩。当 Group Replication 的 Group Communication System（GCS）接收到一个由`group_replication_transaction_size_limit`设置允许但超过`group_replication_compression_threshold`设置的消息时，将执行压缩。更多信息，请参见第 20.7.4 节，“消息压缩”。

+   使用系统变量`group_replication_communication_max_message_size`来指定应用分段的消息大小阈值。该系统变量默认为 10485760 字节（10 MiB），因此大消息会自动进行分段。如果压缩后的消息仍超过`group_replication_communication_max_message_size`限制，GCS 将在压缩后执行分段。为了使复制组使用分段，所有组成员必须使用 MySQL 8.0.16 或更高版本，并且组使用的 Group Replication 通信协议版本必须允许分段。更多信息，请参见第 20.7.5 节，“消息分段”。

最大交易大小、消息压缩和消息分段都可以通过指定相关系统变量的零值来停用。如果您已停用了所有这些保护措施，那么在复制组成员的应用程序线程可以处理的消息的上限大小是成员的`replica_max_allowed_packet`或`slave_max_allowed_packet`系统变量的值，这些变量的默认和最大值为 1073741824 字节（1 GB）。当接收成员尝试处理超过此限制的消息时，消息将失败。组成员可以发起并尝试传输到组的消息的上限大小为 4294967295 字节（约 4 GB）。这是接受由组复制（XCom，一种 Paxos 变体）的组通信引擎处理后的消息的数据包大小的硬限制，GCS 在处理消息后接收它们。当发起成员尝试广播超过此限制的消息时，消息将失败。
