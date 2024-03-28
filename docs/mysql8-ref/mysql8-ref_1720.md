> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-logs-event-buffer.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-logs-event-buffer.html)

#### 25.6.2.3 集群日志中的事件缓冲区报告

`NDB`使用一个或多个内存缓冲区来存储从数据节点接收的事件。每个订阅表事件的[`Ndb`](https://dev.mysql.com/doc/refman/8.0/en/ndbapi/en/ndb-ndb.html)对象都有一个这样的缓冲区，这意味着每个执行二进制日志记录的**mysqld**通常有两个缓冲区（一个用于模式事件，一个用于数据事件）。每个缓冲区包含由事件组成的时代。这些事件包括操作类型（插入、更新、删除）和行数据（前后图像加元数据）。

`NDB`在集群日志中生成消息来描述这些缓冲区的状态。尽管这些报告出现在集群日志中，但它们是指 API 节点上的缓冲区（与大多数其他集群日志消息不同，这些消息是由数据节点生成的）。

集群日志中的事件缓冲区记录使用以下格式：

```sql
Node *node_id*: Event buffer status (*object_id*):
used=*bytes_used* (*percent_used*% of alloc)
alloc=*bytes_allocated* (*percent_alloc*% of max) max=*bytes_available*
latest_consumed_epoch=*latest_consumed_epoch*
latest_buffered_epoch=*latest_buffered_epoch*
report_reason=*report_reason*
```

组成此报告的字段在此处列出，并附有描述：

+   *`node_id`*: 报告来源节点的 ID。

+   *`object_id`*: 报告来源的[`Ndb`](https://dev.mysql.com/doc/refman/8.0/en/ndbapi/en/ndb-ndb.html)对象的 ID。

+   *`bytes_used`*: 缓冲区使用的字节数。

+   *`percent_used`*: 使用的分配字节的百分比。

+   *`bytes_allocated`*: 分配给此缓冲区的字节数。

+   *`percent_alloc`*: 使用的可用字节的百分比；如果[`ndb_eventbuffer_max_alloc`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-options-variables.html#sysvar_ndb_eventbuffer_max_alloc)等于 0（无限制），则不打印。

+   *`bytes_available`*: 可用字节数；如果`ndb_eventbuffer_max_alloc`为 0（无限制），则为 0。

+   *`latest_consumed_epoch`*: 最近完全消耗的时代。（在 NDB API 应用程序中，通过调用[`nextEvent()`](https://dev.mysql.com/doc/refman/8.0/en/ndbapi/en/ndb-ndb.html#ndb-ndb-nextevent)来完成。）

+   *`latest_buffered_epoch`*: 最近在事件缓冲区中缓冲的时代。

+   *`report_reason`*: 报告原因。可能的原因稍后在本节中显示。

报告原因的可能原因在以下列表中描述：

+   `ENOUGH_FREE_EVENTBUFFER`：事件缓冲��有足够的空间。

    `LOW_FREE_EVENTBUFFER`：事件缓冲区的剩余空间不足。

    触发这些报告的阈值空闲百分比水平可以通过设置[`ndb_report_thresh_binlog_mem_usage`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-options-variables.html#sysvar_ndb_report_thresh_binlog_mem_usage)服务器变量来调整。

+   `BUFFERED_EPOCHS_OVER_THRESHOLD`: 缓存的时代数量是否超过了配置的阈值。这个数字是已完整接收的最新时代与最近被消耗的时代之间的差异（在 NDB API 应用程序中，通过调用`nextEvent()`或`nextEvent2()`来完成）。报告每秒生成一次，直到缓存的时代数量低于阈值，可以通过设置`ndb_report_thresh_binlog_epoch_slip`服务器变量来调整阈值。您还可以通过调用`setEventBufferQueueEmptyEpoch()`在 NDB API 应用程序中调整阈值。

+   `PARTIALLY_DISCARDING`: 事件缓冲内存已经耗尽，即已使用了 100%的`ndb_eventbuffer_max_alloc`。任何部分缓存的时代都会被完全缓存，即使使用率超过 100%，但任何接收到的新时代都会被丢弃。这意味着事件流中出现了间隙。

+   `COMPLETELY_DISCARDING`: 不会缓存任何时代。

+   `PARTIALLY_BUFFERING`: 在间隙之后，缓冲区的空闲百分比已经上升到阈值，在**mysql**客户端中可以使用`ndb_eventbuffer_free_percent`服务器系统变量或在 NDB API 应用程序中通过调用`set_eventbuffer_free_percent()`来设置阈值。新的时代被缓存。由于间隙而无法完成的时代被丢弃。

+   `COMPLETELY_BUFFERING`: 所有接收到的时代都正在被缓存，这意味着有足够的事件缓冲内存。事件流中的间隙已经被关闭。
