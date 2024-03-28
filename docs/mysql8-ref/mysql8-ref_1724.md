> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-log-events.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-log-events.html)

#### 25.6.3.2 NDB 集群日志事件

事件日志中报告的事件报告具有以下格式：

```sql
*datetime* [*string*] *severity* -- *message*
```

例如：

```sql
09:19:30 2005-07-24 [NDB] INFO -- Node 4 Start phase 4 completed
```

本节讨论按类别和每个类别内的严重级别排序的所有可报告事件。

在事件描���中，GCP 和 LCP 分别表示“全局检查点”和“本地检查点”。

##### 连接事件

这些事件与集群节点之间的连接相关联。

**表 25.56 与集群节点之间连接相关的事件**

| 事件 | 优先级 | 严重级别 | 描述 |
| --- | --- | --- | --- |
| `Connected` | 8 | `INFO` | 数据节点已连接 |
| `Disconnected` | 8 | `ALERT` | 数据节点断开连接 |
| `CommunicationClosed` | 8 | `INFO` | SQL 节点或数据节点连接已关闭 |
| `CommunicationOpened` | 8 | `INFO` | SQL 节点或数据节点连接已打开 |
| `ConnectedApiVersion` | 8 | `INFO` | 使用 API 版本连接 |

##### 检查点事件

此处显示的日志消息与检查点相关。

**表 25.57 与检查点相关的事件**

| 事件 | 优先级 | 严重级别 | 描述 |
| --- | --- | --- | --- |
| `GlobalCheckpointStarted` | 9 | `INFO` | GCP 开始：REDO 日志写入磁盘 |
| `GlobalCheckpointCompleted` | 10 | `INFO` | GCP 完成 |
| `LocalCheckpointStarted` | 7 | `INFO` | LCP 开始：数据写入磁盘 |
| `LocalCheckpointCompleted` | 7 | `INFO` | LCP 正常完成 |
| `LCPStoppedInCalcKeepGci` | 0 | `ALERT` | LCP 停止 |
| `LCPFragmentCompleted` | 11 | `INFO` | 片段上的 LCP 已完成 |
| `UndoLogBlocked` | 7 | `INFO` | UNDO 日志记录被阻止；缓冲区接近溢出 |
| `RedoStatus` | 7 | `INFO` | 重做状态 |

##### 启动事件

以下事件是响应节点或集群启动及其成功或失败而生成的。它们还提供与启动过程的进展相关的信息，包括有关日志活动的信息。

**表 25.58 与节点或集群启动相关的事件**

| 事件 | 优先级 | 严重级别 | 描述 |
| --- | --- | --- | --- |
| `NDBStartStarted` | 1 | `INFO` | 数据节点启动阶段已启动（所有节点启动） |
| `NDBStartCompleted` | 1 | `INFO` | 启动阶段完成，所有数据节点 |
| `STTORRYRecieved` | 15 | `INFO` | 重新启动完成后接收的块 |
| `StartPhaseCompleted` | 4 | `INFO` | 数据节点启动阶段 *`X`* 完成 |
| `CM_REGCONF` | 3 | `INFO` | 节点已成功包含在集群中；显示节点、管理节点和动态 ID |
| `CM_REGREF` | 8 | `INFO` | 节点被拒绝加入集群；由于配置错误、无法建立通信或其他问题，无法加入集群 |
| `FIND_NEIGHBOURS` | 8 | `INFO` | 显示相邻数据节点 |
| `NDBStopStarted` | 1 | `INFO` | 数据节点关闭已启动 |
| `NDBStopCompleted` | 1 | `INFO` | 数据节点关闭完成 |
| `NDBStopForced` | 1 | `ALERT` | 强制关闭数据节点 |
| `NDBStopAborted` | 1 | `INFO` | 无法正常关闭数据节点 |
| `StartREDOLog` | 4 | `INFO` | 新重做日志已开始；GCI 保持 *`X`*，最新可恢复的 GCI *`Y`* |
| `StartLog` | 10 | `INFO` | 新日志已开始；日志部分 *`X`*，起始 MB *`Y`*，结束 MB *`Z`* |
| `UNDORecordsExecuted` | 15 | `INFO` | 撤销记录已执行 |
| `StartReport` | 4 | `INFO` | 报告已开始 |
| `LogFileInitStatus` | 7 | `INFO` | 日志文件初始化状态 |
| `LogFileInitCompStatus` | 7 | `INFO` | 日志文件完成状态 |
| `StartReadLCP` | 10 | `INFO` | 开始读取本地检查点 |
| `ReadLCPComplete` | 10 | `INFO` | 读取本地检查点已完成 |
| `RunRedo` | 8 | `INFO` | 运行重做日志 |
| `RebuildIndex` | 10 | `INFO` | 重建索引 |
| 事件 | 优先级 | 严重级别 | 描述 |

##### NODERESTART 事件

当重新启动节点并与节点重新启动过程的成功或失败相关时，将生成以下事件。

**表 25.59 与重新启动节点相关的事件**

| 事件 | 优先级 | 严重级别 | 描述 |
| --- | --- | --- | --- |
| `NR_CopyDict` | 7 | `INFO` | 完成复制字典信息 |
| `NR_CopyDistr` | 7 | `INFO` | 完成复制分布信息 |
| `NR_CopyFragsStarted` | 7 | `INFO` | 开始复制片段 |
| `NR_CopyFragDone` | 10 | `INFO` | 完成复制片段 |
| `NR_CopyFragsCompleted` | 7 | `INFO` | 完成复制所有片段 |
| `NodeFailCompleted` | 8 | `ALERT` | 节点故障阶段已完成 |
| `NODE_FAILREP` | 8 | `ALERT` | 报告节点故障 |

| `ArbitState` | 6 | `INFO` | 报告是否找到仲裁者；在寻找仲裁者时有七种不同的可能结果，列在此处：

+   管理服务器重新启动仲裁线程 [状态=*`X`*]

+   准备仲裁节点 *`X`* [票=*`Y`*]

+   接收仲裁节点 *`X`* [票=*`Y`*]

+   启动仲裁节点 *`X`* [票=*`Y`*]

+   丢失仲裁节点 *`X`* - 进程失败 [状态=*`Y`*]

+   丢失仲裁节点 *`X`* - 进程退出 [状态=*`Y`*]

+   丢失仲裁���点 *`X`* <错误消息> [状态=*`Y`*]

|

| `ArbitResult` | 2 | `ALERT` | 报告仲裁结果；仲裁尝试有八种不同的可能结果，列在此处：

+   仲裁检查失败：少于 1/2 的节点剩余

+   仲裁检查成功：节点组多数

+   仲裁检查失败：缺少节点组

+   网络分区：需要仲裁

+   仲裁成功：来自节点 *`X`* 的肯定回应

+   仲裁失败：来自节点 *`X`* 的负面回应

+   网络分区：没有可用的仲裁节点

+   网络分区：未配置仲裁节点

|

| `GCP_TakeoverStarted` | 7 | `INFO` | GCP 接管已启动 |
| --- | --- | --- | --- |
| `GCP_TakeoverCompleted` | 7 | `INFO` | GCP 接管已完成 |
| `LCP_TakeoverStarted` | 7 | `INFO` | LCP 接管已启动 |
| `LCP_TakeoverCompleted` | 7 | `INFO` | LCP 接管完成（状态 = *`X`*） |
| `ConnectCheckStarted` | 6 | `INFO` | 连接检查已启动 |
| `ConnectCheckCompleted` | 6 | `INFO` | 连接检查已完成 |
| `NodeFailRejected` | 6 | `ALERT` | 节点故障阶段失败 |
| 事件 | 优先级 | 严重级别 | 描述 |

##### 统计事件

以下事件具有统计性质。它们提供诸如事务数和其他操作数、单个节点发送或接收的数据量以及内存使用情况等信息。

**表 25.60 统计性质事件**

| 事件 | 优先级 | 严重级别 | 描述 |
| --- | --- | --- | --- |
| `TransReportCounters` | 8 | `INFO` | 报告事务统计信息，包括事务数、提交数、读取数、简单读取数、写入数、并发操作、属性信息和中止数 |
| `OperationReportCounters` | 8 | `INFO` | 操作数 |
| `TableCreated` | 7 | `INFO` | 报告创建的表数 |
| `JobStatistic` | 9 | `INFO` | 平均内部作业调度统计 |
| `ThreadConfigLoop` | 9 | `INFO` | 线程配置循环数 |
| `SendBytesStatistic` | 9 | `INFO` | 发送到节点 *`X`* 的平均字节数 |
| `ReceiveBytesStatistic` | 9 | `INFO` | 从节点 *`X`* 接收的平均字节数 |
| `MemoryUsage` | 5 | `INFO` | 数据和索引内存使用情况（80%、90%和 100%） |
| `MTSignalStatistics` | 9 | `INFO` | 多线程信号 |

##### 模式事件

这些事件涉及 NDB 集群模式操作。

**表 25.61 与 NDB 集群模式操作相关的事件**

| 事件 | 优先级 | 严重级别 | 描述 |
| --- | --- | --- | --- |
| `CreateSchemaObject` | 8 | `INFO` | 创建模式对象 |
| `AlterSchemaObject` | 8 | `INFO` | 模式对象已更新 |
| `DropSchemaObject` | 8 | `INFO` | 模式对象已删除 |

##### 错误事件

这些事件涉及集群错误和警告。其中一个或多个的存在通常表示发生了重大故障或失败。

**表 25.62 与集群错误和警告相关的事件**

| 事件 | 优先级 | 严重级别 | 描述 |
| --- | --- | --- | --- |
| `TransporterError` | 2 | `ERROR` | 传输器错误 |
| `TransporterWarning` | 8 | `WARNING` | 传输器警告 |
| `MissedHeartbeat` | 8 | `WARNING` | 节点 *`X`* 错过心跳次数 *`Y`* |
| `DeadDueToHeartbeat` | 8 | `ALERT` | 节点 *`X`* 因错过心跳被宣布“死亡” |
| `WarningEvent` | 2 | `WARNING` | 一般警告事件 |
| `SubscriptionStatus` | 4 | `WARNING` | 订阅状态变更 |

##### 信息事件

这些事件提供有关集群状态和与集群维护相关活动的一般信息，例如日志记录和心跳传输。

**表 25.63 信息事件**

| 事件 | 优先级 | 严重级别 | 描述 |
| --- | --- | --- | --- |
| `SentHeartbeat` | 12 | `INFO` | 发送心跳 |
| `CreateLogBytes` | 11 | `INFO` | 创建日志：日志部分、日志文件、大小（MB） |
| `InfoEvent` | 2 | `INFO` | 一般信息事件 |
| `EventBufferStatus` | 7 | `INFO` | 事件缓冲区状态 |
| `EventBufferStatus2` | 7 | `INFO` | 改进的事件缓冲区状态信息 |

注意

`SentHeartbeat` 事件仅在使用 `VM_TRACE` 编译的 NDB Cluster 中可用。

##### 单用户事件

这些事件与进入和退出单用户模式相关联。

**表 25.64 与单用户模式相关的事件**

| 事件 | 优先级 | 严重级别 | 描述 |
| --- | --- | --- | --- |
| `SingleUser` | 7 | `INFO` | 进入或退出单用户模式 |

##### 备份事件

这些事件提供有关创建或恢复备份的信息。

**表 25.65 备份事件**

| 事件 | 优先级 | 严重级别 | 描述 |
| --- | --- | --- | --- |
| `BackupStarted` | 7 | `INFO` | 备份开始 |
| `BackupStatus` | 7 | `INFO` | 备份状态 |
| `BackupCompleted` | 7 | `INFO` | 备份完成 |
| `BackupFailedToStart` | 7 | `ALERT` | 备份启动失败 |
| `BackupAborted` | 7 | `ALERT` | 用户中止备份 |
| `RestoreStarted` | 7 | `INFO` | 开始从备份恢复 |
| `RestoreMetaData` | 7 | `INFO` | 恢复元数据 |
| `RestoreData` | 7 | `INFO` | 恢复数据 |
| `RestoreLog` | 7 | `INFO` | 恢复日志文件 |
| `RestoreCompleted` | 7 | `INFO` | 备份恢复完成 |
| `SavedEvent` | 7 | `INFO` | 事件已保存 |
| 事件 | 优先级 | 严重级别 | 描述 |
