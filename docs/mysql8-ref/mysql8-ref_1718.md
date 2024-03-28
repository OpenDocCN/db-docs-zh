> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-logs-cluster-log.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-logs-cluster-log.html)

#### 25.6.2.1 NDB 集群：集群日志中的消息

以下表列出了最常见的 `NDB` 集群日志消息。有关集群日志、日志事件和事件类型的信息，请参阅 第 25.6.3 节 “NDB 集群中生成的事件报告”。这些日志消息还对应于 MGM API 中的日志事件类型；有关集群 API 开发人员感兴趣的相关信息，请参阅 Ndb_logevent_type 类型。

**表 25.53 常见 NDB 集群日志消息**

| 日志消息 | 描述 | 事件名称 | 事件类型 | 优先级 | 严重性 |
| --- | --- | --- | --- | --- | --- |
| `Node *`mgm_node_id`*: 节点 *`data_node_id`* 已连接` | 具有节点 ID *`node_id`* 的数据节点已连接到管理服务器（节点 *`mgm_node_id`*）。 | `已连接` | `连接` | 8 | `信息` |
| `Node *`mgm_node_id`*: 节点 *`data_node_id`* 已断开连接` | 具有节点 ID *`data_node_id`* 的数据节点已从管理服务器（节点 *`mgm_node_id`*）断开连接。 | `已断开连接` | `连接` | 8 | `警报` |
| `Node *`data_node_id`*: 与节点 *`api_node_id`* 的通信已关闭` | 具有节点 ID *`api_node_id`* 的 API 节点或 SQL 节点不再与数据节点 *`data_node_id`* 进行通信。 | `通信已关闭` | `连接` | 8 | `信息` |
| `Node *`data_node_id`*: 与节点 *`api_node_id`* 的通信已打开` | 具有节点 ID *`api_node_id`* 的 API 节点或 SQL 节点现在正在与数据节点 *`data_node_id`* 进行通信。 | `通信已打开` | `连接` | 8 | `信息` |
| `Node *`mgm_node_id`*: 节点 *`api_node_id`*：API *`version`*` | 具有节点 ID *`api_node_id`* 的 API 节点已连接到管理节点 *`mgm_node_id`*，使用 `NDB` API 版本 *`version`*（通常与 MySQL 版本号相同）。 | `ConnectedApiVersion` | `连接` | 8 | `信息` |
| `Node *`node_id`*: 全局检查点 *`gci`* 已启动` | 具有 ID *`gci`* 的全局检查点已启动；节点 *`node_id`* 是负责此全局检查点的主节点。 | `全局检查点已启动` | `检查点` | 9 | `信息` |
| `Node *`node_id`*: 全局检查点 *`gci`* 已完成` | 具有 ID *`gci`* 的全局检查点已完成；节点 *`node_id`* 是负责此全局检查点的主节点。 | `全局检查点已完成` | `检查点` | 10 | `信息` |
| `Node *`node_id`*: 本地检查点*`lcp`*已启动。保持 GCI = *`current_gci`*，最旧可恢复的 GCI = *`old_gci`*` | 具有序列 ID *`lcp`* 的本地检查点已在节点 *`node_id`* 上启动。可以使用的最近的 GCI 具有索引 *`current_gci`*，而集群可以恢复的最旧 GCI 具有索引 *`old_gci`*。 | `LocalCheckpointStarted` | `检查点` | 7 | `信息` |
| `Node *`node_id`*: 本地检查点*`lcp`*已完成` | 在节点 *`node_id`* 上具有序列 ID *`lcp`* 的本地检查点已经完成。 | `LocalCheckpointCompleted` | `检查点` | 8 | `信息` |
| `Node *`node_id`*: 本地检查点在 CALCULATED_KEEP_GCI 中停止` | 节点无法确定最近可用的 GCI。 | `LCPStoppedInCalcKeepGci` | `检查点` | 0 | `警报` |
| `Node *`node_id`*: 表 ID = *`table_id`*，片段 *`ID = fragment_id`* 在节点 *`node_id`* 上完成了 LCP，maxGciStarted: *`started_gci`* maxGciCompleted: *`completed_gci`*` | 一个表片段已经在节点 *`node_id`* 上被检查点到磁盘上。正在进行的 GCI 具有索引 *`started_gci`*，而最近完成的 GCI 具有索引 *`completed_gci`*。 | `LCPFragmentCompleted` | `检查点` | 11 | `信息` |
| `Node *`node_id`*: ACC 阻塞了*`num_1`*次，TUP 阻塞了*`num_2`*次上一秒` | 因为日志缓冲区接近溢出，撤销日志记录被阻塞。 | `UndoLogBlocked` | `检查点` | 7 | `信息` |
| `Node *`node_id`*: 启动初始化*`version`*` | 数据节点 *`node_id`*，运行`NDB`版本 *`version`*，正在开始启动过程。 | `NDBStartStarted` | `启动` | 1 | `信息` |
| `Node *`node_id`*: 启动*`version`*已启动` | 运行`NDB`版本 *`version`* 的数据节点 *`node_id`* 已成功启动。 | `NDBStartCompleted` | `启动` | 1 | `信息` |
| `Node *`node_id`*: 重启完成后收到 STTORRY 信号` | 节点收到一个信号，表示集群重启已经完成。 | `STTORRYRecieved` | `启动` | 15 | `信息` |
| `Node *`node_id`*: 启动阶段*`phase`*已完成（*`type`*）` | 节点已完成*`type`*启动的启动阶段*`phase`*。有关启动阶段的列表，请参见第 25.6.4 节，“NDB 集群启动阶段总结”。(*`type`*是`initial`、`system`、`node`、`initial node`或`<未知>`之一。) | `StartPhaseCompleted` | `启动` | 4 | `信息` |
| `Node *`node_id`*: CM_REGCONF 主席 = *`president_id`*，自己的节点 = *`own_id`*，我们的动态 ID = *`dynamic_id`*` | 节点 *`president_id`* 被选为“主席”。*`own_id`* 和 *`dynamic_id`* 应始终与报告节点的 ID（*`node_id`*）相同。 | `CM_REGCONF` | `启动` | 3 | `信息` |
| `Node *`node_id`*: 来自节点 *`president_id`* 对我们节点 *`node_id`* 的 CM_REGREF。原因 = *`cause`*` | 报告节点（ID *`node_id`*）无法接受节点 *`president_id`* 为主席。问题的*`cause`* 给出为`Busy`、`Election with wait = false`、`Not president`、`Election without selecting new candidate`或`No such cause`之一。 | `CM_REGREF` | `StartUp` | 8 | `INFO` |
| `Node *`node_id`*: 我们是节点 *`own_id`*，动态 ID 为 *`dynamic_id`*，左邻节点是节点 *`id_1`*，右邻节点是节点 *`id_2`*` | 节点已发现其在集群中的邻居节点（节点 *`id_1`* 和节点 *`id_2`*）。*`node_id`*、*`own_id`* 和 *`dynamic_id`* 应始终相同；如果它们不同，则表示集群节点配置严重错误。 | `FIND_NEIGHBOURS` | `StartUp` | 8 | `INFO` |
| `Node *`node_id`*: *`type`* 关闭已启动` | 节点已接收到关闭信号。关闭的*`type`*可以是`Cluster`或`Node`。 | `NDBStopStarted` | `StartUp` | 1 | `INFO` |
| `Node *`node_id`*: 节点关闭已完成` [`, *`action`*`] [`由信号 *`signal`* 发起。] | 节点已关闭。此报告可能包括一个*`action`*，如果存在，则为`restarting`、`no start`或`initial`之一。报告还可能包括对`NDB`协议 *`signal`* 的引用；有关可能的信号，请参阅操作和信号。 | `NDBStopCompleted` | `StartUp` | 1 | `INFO` |
| `Node *`node_id`*: 强制节点关闭已完成` [`, action`]`.` [`发生在启动阶段 *`start_phase`*。] [`由 *`signal`* 发起。] [`由错误 *`error_code`*: '*`error_message`*(*`error_classification`*). *`error_status`*'.` [`(额外信息 *`extra_code`*)`]] | 节点已被强制关闭。随后采取的*`action`*（`restarting`、`no start`或`initial`之一）也会报告。如果节点在启动时关闭，报告将包括节点失败的*`start_phase`*。如果这是由发送给节点的*`signal`* 导致的，还会提供此信息（有关更多信息，请参阅操作和信号）。如果已知导致失败的错误，也会包括在内；有关更多关于`NDB`错误消息和分类的信息，请参阅 NDB Cluster API 错误。 | `NDBStopForced` | `StartUp` | 1 | `ALERT` |
| `Node *`node_id`*: 节点关闭已中止` | 用户中止了节点关闭过程。 | `NDBStopAborted` | `StartUp` | 1 | `INFO` |
| `节点 *`node_id`*：StartLog：[GCI 保留：*`keep_pos`* 最后完成：*`last_pos`* 最新可恢复：*`restore_pos`*]` | 这报告了节点启动期间引用的全局检查点。在 *`keep_pos`* 之前的重做日志将被丢弃。*`last_pos`* 是数据节点参与的最后一个全局检查点；*`restore_pos`* 是实际用于恢复所有数据节点的全局检查点。 | `StartREDOLog` | `StartUp` | 4 | `信息` |
| *`startup_message`* [*单独列出；请参见下文.*] | 在不同情况下可能记录的启动消息有很多种。这些将单独列出；请参见第 25.6.2.2 节，“NDB Cluster 日志启动消息”。 | `StartReport` | `StartUp` | 4 | `信息` |
| `节点 *`node_id`*：节点重新启动完成了字典信息的复制` | 数据字典信息已复制到重新启动的节点 | `NR_CopyDict` | `NodeRestart` | 8 | `信息` |
| `节点 *`node_id`*：节点重新启动完成了分布信息的复制` | 数据分布信息已复制到重新启动的节点 | `NR_CopyDistr` | `NodeRestart` | 8 | `信息` |
| `节点 *`node_id`*：节点重新启动开始复制片段到节点 *`node_id`*` | 开始将片段复制到正在重新启动的数据节点 *`node_id`* | `NR_CopyFragsStarted` | `NodeRestart` | 8 | `信息` |
| `Node *`node_id`*：表 ID = *`table_id`*，片段 ID = *`fragment_id`* 已复制到节点 *`node_id`*` | 来自表 *`table_id`* 的片段 *`fragment_id`* 已复制到数据节点 *`node_id`* | `NR_CopyFragDone` | `NodeRestart` | 10 | `信息` |
| `节点 *`node_id`*：节点重新启动完成了复制片段到节点 *`node_id`*` | 将所有表片段复制到重新启动的数据节点 *`node_id`* 已完成 | `NR_CopyFragsCompleted` | `NodeRestart` | 8 | `信息` |
| `Node *`node_id`*：节点 *`node1_id`* 完成了节点 *`node2_id`* 的故障` | 数据节点 *`node1_id`* 检测到了数据节点 *`node2_id`* 的故障 | `NodeFailCompleted` | `NodeRestart` | 8 | `警报` |
| `所有节点完成了节点 *`node_id`* 的故障` | 所有（剩余）数据节点已检测到数据节点 *`node_id`* 的故障 | `NodeFailCompleted` | `NodeRestart` | 8 | `警报` |
| `节点 *`node_id`* 的块故障已完成` | 在`NDB`内核块中检测到了数据节点 *`node_id`* 的故障，其中块是 `DBTC` 的第 1 块，`DBDICT`，`DBDIH`，或 `DBLQH`；更多信息，请参阅 NDB 内核块 | `NodeFailCompleted` | `NodeRestart` | 8 | `警报` |
| `节点 *`mgm_node_id`*: 节点 *`data_node_id`* 失败。节点在失败时的状态为 *`state_code`*` | 一个数据节点失败了。其在失败时的状态由仲裁状态代码 *`state_code`* 描述：可能的状态代码值可以在文件`include/kernel/signaldata/ArbitSignalData.hpp`中找到。 | `节点失败报告` | `节点重启` | 8 | `警报` |
| `主节点重新启动仲裁线程 [状态=*`state_code`*]` 或 `准备仲裁节点 *`node_id`* [票=*`ticket_id`*]` 或 `接收仲裁节点 *`node_id`* [票=*`ticket_id`*]` 或 `启动仲裁节点 *`node_id`* [票=*`ticket_id`*]` 或 `丢失仲裁节点 *`node_id`* - 进程失败 [状态=*`state_code`*]` 或 `丢失仲裁节点 *`node_id`* - 进程退出 [状态=*`state_code`*]` 或 `丢失仲裁节点 *`node_id`* - *`error_message`* [状态=*`state_code`*]` | 这是对集群中仲裁当前状态和进展的报告。*`node_id`*是选定为仲裁者的管理节点或 SQL 节点的节点 ID。*`state_code`*是一个仲裁状态代码，如`include/kernel/signaldata/ArbitSignalData.hpp`中所述。当发生错误时，还会提供一个*`error_message`*，也在`ArbitSignalData.hpp`中定义。*`ticket_id`*是在选定为仲裁者时由仲裁者分发给所有参与其选择的节点的唯一标识符；这用于确保请求仲裁的每个节点都是参与选择过程的节点之一。 | `仲裁状态` | `节点重启` | 6 | `信息` |
| `仲裁检查失败 - 不到 1/2 节点剩余` 或 `仲裁检查成功 - 所有节点组和超过 1/2 节点剩余` 或 `仲裁检查成功 - 节点组多数` 或 `仲裁检查失败 - 缺少节点组` 或 `网络分区 - 需要仲裁` 或 `仲裁成功 - 节点 *`node_id`*` 或 `仲裁失败 - 节点 *`node_id`*` 或 `网络分区 - 没有仲裁者可用` 或 `网络分区 - 没有配置仲裁者` 或 `仲裁失败 - *`error_message`* [状态=*`state_code`*]` | 此消息报告了仲裁的结果。在仲裁失败的情况下，会提供一个*`error_message`*和一个仲裁*`state_code`*；这两者的定义可以在`include/kernel/signaldata/ArbitSignalData.hpp`中找到。 | `仲裁结果` | `节点重启` | 2 | `警报` |
| `节点 *`node_id`*: GCP 接管开始` | 此节点正在尝试承担下一个全局检查点的责任（即，它正在成为主节点） | `GCP_ 接管开始` | `节点重启` | 7 | `信息` |
| `节点 *`node_id`*: GCP 接管完成` | 此节点已成为主节点，并已承担下一个全局检查点的责任 | `GCP_ 接管完成` | `节点重启` | 7 | `信息` |
| `Node *`node_id`*: LCP 接管已启动` | 该节点正在尝试承担下一组本地检查点的责任（即成为主节点） | `LCP_TakeoverStarted` | `NodeRestart` | 7 | `INFO` |
| `Node *`node_id`*: LCP 接管已完成` | 该节点已成为主节点，并已承担下一组本地检查点的责任 | `LCP_TakeoverCompleted` | `NodeRestart` | 7 | `INFO` |
| `Node *`node_id`*: 事务计数 = *`transactions`*，提交计数 = *`commits`*，读取计数 = *`reads`*，简单读取计数 = *`simple_reads`*，写入计数 = *`writes`*，AttrInfo 计数 = *`AttrInfo_objects`*，并发操作 = *`concurrent_operations`*，中止计数 = *`aborts`*，扫描 = *`scans`*，范围扫描 = *`range_scans`*` | 大约每 10 秒提供一次的事务活动报告 | `TransReportCounters` | `Statistic` | 8 | `INFO` |
| `Node *`node_id`*: Operations=*`operations`*` | 该节点执行的操作数量，大约每 10 秒提供一次 | `OperationReportCounters` | `Statistic` | 8 | `INFO` |
| `Node *`node_id`*: ID 为 *`table_id`* 的表已创建` | 已创建具有显示的表 ID 的表 | `TableCreated` | `Statistic` | 7 | `INFO` |
| `Node *`node_id`*: 在 doJob 中最后 8192 次的平均循环计数 = *`count`*` |  | `JobStatistic` | `Statistic` | 9 | `INFO` |
| `平均发送大小到节点 = *`node_id`* 最后 4096 次发送 = *`bytes`* 字节` | 该节点每次发送到节点 *`node_id`* 的平均发送 *`bytes`* 字节 | `SendBytesStatistic` | `Statistic` | 9 | `INFO` |
| `平均接收大小到节点 = *`node_id`* 最后 4096 次发送 = *`bytes`* 字节` | 每次从节点 *`node_id`* 接收数据时，该节点平均接收 *`bytes`* 字节的数据 | `ReceiveBytesStatistic` | `Statistic` | 9 | `INFO` |
| `Node *`node_id`*: 数据使用率为 *`data_memory_percentage`*%（*`data_pages_used`* 32K 页，总共 *`data_pages_total`*）` / `Node *`node_id`*: 索引使用率为 *`index_memory_percentage`*%（*`index_pages_used`* 8K 页，总共 *`index_pages_total`*）` | 当在集群管理客户端中发出 `DUMP 1000` 命令时生成此报告 | `MemoryUsage` | `Statistic` | 5 | `INFO` |
| `Node *`node1_id`*: 到节点 *`node2_id`* 的传输器报告错误 *`error_code`*: *`error_message`*` | 与节点 *`node2_id`* 通信时发生传输器错误；有关传输器错误代码和消息的列表，请参阅 NDB 传输器错误，在 MySQL NDB 集群内部手册中 | `TransporterError` | `Error` | 2 | `ERROR` |
| `Node *`node1_id`*: 传输到节点 *`node2_id`* 报告错误 *`error_code`*: *`error_message`*` | 在与节点 *`node2_id`* 通信时出现潜在传输器问题的警告；有关传输器错误代码和消息的列表，请参见 NDB 传输器错误，有关更多信息 |
| `Node *`node1_id`*: 节点 *`node2_id`* 丢失心跳 *`heartbeat_id`*` | 这个节点丢失了来自节点 *`node2_id`* 的心跳 |
| `Node *`node1_id`*: 节点 *`node2_id`* 因丢失心跳而被宣告死亡` | 这个节点已经至少丢失了来自节点 *`node2_id`* 的 3 个心跳，因此宣告该节点“死亡” |
| `Node *`node1_id`*: 向节点发送心跳 = *`node2_id`*` | 这个节点已向节点 *`node2_id`* 发送了心跳 |
| `Node *`node_id`*: 事件缓冲区状态 (*`object_id`*): 已使用=*`bytes_used`* (*`percent_used`*% of alloc) 分配=*`bytes_allocated`* 最大=*`bytes_available`* 最新消耗的时代=*`latest_consumed_epoch`* 最新缓冲的时代=*`latest_buffered_epoch`* 报告原因=*`report_reason`*` | 在事件缓冲区使用量较大时出现此报告，例如，在相对较短的时间内应用了许多更新时；报告显示了已使用的事件缓冲区内存的字节数和百分比，仍然可用的字节数和百分比，以及最新的缓冲和消耗时代；有关更多信息，请参见第 25.6.2.3 节，“集群日志中的事件缓冲区报告” |
| `Node *`node_id`*: 进入单用户模式`, `Node *`node_id`*: 进入单用户模式 节点 *`API_node_id`* 具有独占访问权限`, `Node *`node_id`*: 进入单用户模式` | 当进入和退出单用户模式时，这些报告将被写入集群日志；*`API_node_id`* 是具有对集群的独占访问权限的 API 或 SQL 的节点 ID（有关更多信息，请参见第 25.6.6 节，“NDB 集群单用户模式”）；消息 `未知单用户报告 *`API_node_id`*` 表示发生了错误，正常情况下不应看到 |
| `Node *`node_id`*: 从节点 *`mgm_node_id`* 开始的备份已启动` | 使用具有 *`mgm_node_id`* 的管理节点启动了备份；当发出`START BACKUP`命令时，此消息也会显示在集群管理客户端中；有关更多信息，请参阅第 25.6.8.2 节，“使用 NDB 集群管理客户端创建备份” | `备份已启动` | `备份` | 7 | `信息` |
| `Node *`node_id`*: 从节点 *`mgm_node_id`* 开始的备份 *`backup_id`* 已完成。StartGCP: *`start_gcp`* StopGCP: *`stop_gcp`* #记录数：*`records`* #日志记录数：*`log_records`* 数据：*`data_bytes`* 字节 日志：*`log_bytes`* 字节` | 具有 ID *`backup_id`* 的备份已完成；有关更多信息，请参阅第 25.6.8.2 节，“使用 NDB 集群管理客户端创建备份” | `备份已完成` | `备份` | 7 | `信息` |
| `Node *`node_id`*: 来自 *`mgm_node_id`* 的备份请求启动失败。错误：*`error_code`*` | 备份启动失败；有关错误代码，请参阅 MGM API 错误 | `备份启动失败` | `备份` | 7 | `警报` |
| `Node *`node_id`*: 从 *`mgm_node_id`* 开始的备份 *`backup_id`* 已中止。错误：*`error_code`*` | 备份在启动后被终止，可能是由于用户干预 | `备份已中止` | `备份` | 7 | `警报` |
| 日志消息 | 描述 | 事件名称 | 事件类型 | 优先级 | 严重程度 |
