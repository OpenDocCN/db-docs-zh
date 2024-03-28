> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-gtids-concepts.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-gtids-concepts.html)

#### 19.1.3.1 GTID 格式和存储

全局事务标识符（GTID）是在源服务器上（源）提交的每个事务创建并关联的唯一标识符。该标识符不仅对于其起源的服务器是唯一的，而且在给定复制拓扑结构中的所有服务器上都是唯一的。

GTID 分配区分了客户端事务（在源上提交）和复制事务（在副本上重现）。当客户端事务在源上提交时，如果事务已写入二进制日志，则会分配一个新的 GTID。客户端事务保证具有单调递增的 GTID，生成的数字之间没有间隙。如果客户端事务未写入二进制日志（例如，因为事务被过滤掉，或者事务是只读的），则在源服务器上不会分配 GTID。

复制事务保留在源服务器上分配给事务的相同 GTID。GTID 在复制事务开始执行之前存在，并且即使在副本上未将复制事务写入二进制日志或在副本上被过滤掉，也会持久存在。MySQL 系统表 `mysql.gtid_executed` 用于保留应用于 MySQL 服务器上的所有事务的分配 GTID，除了那些存储在当前活动二进制日志文件中的事务。

GTID 的自动跳过功能意味着在源上提交的事务在副本上最多只能应用一次，这有助于保证一致性。一旦具有特定 GTID 的事务在给定服务器上提交，那么该服务器将忽略执行具有相同 GTID 的后续事务的任何尝试。不会引发错误，也不会执行事务中的任何语句。

如果具有特定 GTID 的事务已经在服务器上开始执行，但尚未提交或回滚，则在服务器上尝试启动具有相同 GTID 的并发事务会被阻塞。服务器既不开始执行并发事务，也不将控制权返回给客户端。一旦第一次尝试的事务提交或回滚，那些在相同 GTID 上阻塞的并发会话可以继续。如果第一次尝试回滚，一个并发会话将继续尝试事务，而任何其他在相同 GTID 上阻塞的并发会话将保持阻塞。如果第一次尝试提交，所有并发会话将停止被阻塞，并自动跳过事务的所有语句。

GTID 表示为一对坐标，用冒号字符（`:`）分隔，如下所示：

```sql
GTID = *source_id*:*transaction_id*
```

*`source_id`* 标识了来源服务器。通常，使用源的 `server_uuid` 来实现此目的。*`transaction_id`* 是由事务在源上提交的顺序确定的序列号。例如，第一个提交的事务的 *`transaction_id`* 为 `1`，在同一来源服务器上提交的第十个事务被分配一个 *`transaction_id`* 为 `10`。在 GTID 中，事务不可能具有 `0` 作为序列号。例如，在具有 UUID `3E11FA47-71CA-11E1-9E33-C80AA9429562` 的服务器上最初提交的第二十三个事务具有以下 GTID：

```sql
3E11FA47-71CA-11E1-9E33-C80AA9429562:23
```

服务器实例上 GTID 的序列号上限是有符号 64 位整数的非负值数量（2 的 63 次方减 1，即 9,223,372,036,854,775,807）。如果服务器用尽了 GTID，将执行 `binlog_error_action` 中指定的操作。从 MySQL 8.0.23 开始，当服务器实例接近限制时会发出警告消息。

事务的 GTID 在 **mysqlbinlog** 的输出中显示，并用于在性能模式复制状态表中标识单个事务，例如 `replication_applier_status_by_worker`。`gtid_next` 系统变量 (`@@GLOBAL.gtid_next`) 存储的是单个 GTID。

##### GTID 集合

一个 GTID 集合由一个或多个单个 GTID 或 GTID 范围组成。在 MySQL 服务器中，GTID 集合有多种用途。例如，`gtid_executed` 和 `gtid_purged` 系统变量存储的值就是 GTID 集合。`START REPLICA`（或 MySQL 8.0.22 之前的 `START SLAVE`）子句 `UNTIL SQL_BEFORE_GTIDS` 和 `UNTIL SQL_AFTER_GTIDS` 可以用来使复制进程仅处理 GTID 集合中第一个 GTID 之前的事务，或在 GTID 集合中最后一个 GTID 之后停止。内置函数 `GTID_SUBSET()` 和 `GTID_SUBTRACT()` 需要 GTID 集合作为输入。

来自同一服务器的一系列 GTID 可以合并为一个表达式，如下所示：

```sql
3E11FA47-71CA-11E1-9E33-C80AA9429562:1-5
```

上述示例代表了在 MySQL 服务器上起源的第一到第五个事务，其`server_uuid`为`3E11FA47-71CA-11E1-9E33-C80AA9429562`。来自同一服务器的多个单个 GTID 或 GTID 范围也可以包含在单个表达式中，GTID 或范围之间用冒号分隔，如下例所示：

```sql
3E11FA47-71CA-11E1-9E33-C80AA9429562:1-3:11:47-49
```

GTID 集可以包含任意组合的单个 GTID 和 GTID 范围，并且可以包含来自不同服务器的 GTID。此示例显示了存储在副本的`gtid_executed`系统变量（`@@GLOBAL.gtid_executed`）中的 GTID 集，该副本已应用来自多个来源的事务：

```sql
2174B383-5441-11E8-B90A-C80AA9429562:1-3, 24DA167-0C0C-11E8-8442-00059A3C7B00:1-19
```

当从服务器变量返回 GTID 集时，UUID 按字母顺序排列，数字间隔被合并并按升序排列。

GTID 集的语法如下：

```sql
*gtid_set*:
    *uuid_set* [, *uuid_set*] ...
    | ''

*uuid_set*:
    *uuid*:*interval*[:*interval*]...

*uuid*:
    *hhhhhhhh*-*hhhh*-*hhhh*-*hhhh*-*hhhhhhhhhhhh*

*h*:
    [0-9|A-F]

*interval*:
    *n*[-*n*]

    (*n* >= 1)
```

##### mysql.gtid_executed 表

GTIDs 存储在名为`gtid_executed`的表中，位于`mysql`数据库中。此表中的一行包含每个 GTID 或其代表的 GTID 集的起始服务器的 UUID，以及集合的起始和结束事务 ID；对于仅引用单个 GTID 的行，这两个值相同。

当安装或升级 MySQL 服务器时，将创建（如果尚不存在）`mysql.gtid_executed`表，使用类似于以下所示的`CREATE TABLE`语句：

```sql
CREATE TABLE gtid_executed (
    source_uuid CHAR(36) NOT NULL,
    interval_start BIGINT(20) NOT NULL,
    interval_end BIGINT(20) NOT NULL,
    PRIMARY KEY (source_uuid, interval_start)
)
```

警告

与其他 MySQL 系统表一样，请不要尝试自行创建或修改此表。

`mysql.gtid_executed`表供 MySQL 服务器内部使用。当在副本上禁用二进制日志记录时，它使副本能够使用 GTIDs，并在二进制日志丢失时保留 GTID 状态。请注意，如果发出`RESET MASTER`命令，`mysql.gtid_executed`表将被清除。

仅当`gtid_mode`为`ON`或`ON_PERMISSIVE`时，GTIDs 才存储在`mysql.gtid_executed`表中。如果二进制日志记录已禁用（`log_bin`为`OFF`），或者如果`log_replica_updates`或`log_slave_updates`已禁用，则服务器将每个事务的 GTID 与事务一起存储在缓冲区中提交事务时，并且后台线程定期将缓冲区的内容作为一个或多个条目添加到`mysql.gtid_executed`表中。此外，表会定期以用户可配置的速率进行压缩，如 mysql.gtid_executed 表压缩中所述。

如果启用了二进制日志记录（`log_bin`为`ON`），从 MySQL 8.0.17 开始，仅适用于`InnoDB`存储引擎，服务器会在事务提交时以与禁用二进制日志记录或复制更新日志相同的方式更新`mysql.gtid_executed`表，存储每个事务的 GTID。然而，在 MySQL 8.0.17 之前的版本中，以及对于其他存储引擎，服务器仅在二进制日志轮换或服务器关闭时更新`mysql.gtid_executed`表。在这些时候，服务器会将之前二进制日志中写入的所有事务的 GTID 写入`mysql.gtid_executed`表。这种情况适用于 MySQL 8.0.17 之前的源端，或者在启用二进制日志记录的 MySQL 8.0.17 之前的复制端，或者使用除`InnoDB`之外的存储引擎，它具有以下后果：

+   在服务器意外停止的情况下，当前二进制日志文件中的 GTID 集合不会保存在`mysql.gtid_executed`表中。这些 GTID 在恢复过程中从二进制日志文件中添加到表中，以便复制可以继续进行。唯一的例外是，如果在服务器重新启动时禁用了二进制日志记录（使用`--skip-log-bin`或`--disable-log-bin`）。在这种情况下，服务器无法访问二进制日志文件以恢复 GTID，因此无法启动复制。

+   `mysql.gtid_executed`表不包含所有已执行事务的 GTID 的完整记录。这些信息由全局值`gtid_executed`系统变量提供。在 MySQL 8.0.17 之前的版本和使用除`InnoDB`之外的存储引擎时，始终使用`@@GLOBAL.gtid_executed`，该值在每次提交后更新，以表示 MySQL 服务器的 GTID 状态，而不是查询`mysql.gtid_executed`表。

MySQL 服务器在只读或超级只读模式下仍可以写入`mysql.gtid_executed`表。在 MySQL 8.0.17 之前的版本中，这确保了在这些模式下仍然可以旋转二进制日志文件。如果无法访问`mysql.gtid_executed`表进行写入，并且二进制日志文件因其他原因而旋转（而不是达到最大文件大小`max_binlog_size`），则继续使用当前的二进制日志文件。向请求旋转的客户端返回错误消息，并在服务器上记录警告。如果无法访问`mysql.gtid_executed`表进行写入，并且达到`max_binlog_size`，则服务器根据其`binlog_error_action`设置做出响应。如果设置为`IGNORE_ERROR`，则在服务器上记录错误并停止二进制日志记录，或者如果设置为`ABORT_SERVER`，则服务器关闭。

##### mysql.gtid_executed 表压缩

随着时间的推移，`mysql.gtid_executed`表可能会填满许多行，这些行引用在同一服务器上起源的单个 GTID，并且其事务 ID 组成一个范围，类似于这里显示的内容：

```sql
+--------------------------------------+----------------+--------------+
| source_uuid                          | interval_start | interval_end |
|--------------------------------------+----------------+--------------|
| 3E11FA47-71CA-11E1-9E33-C80AA9429562 | 37             | 37           |
| 3E11FA47-71CA-11E1-9E33-C80AA9429562 | 38             | 38           |
| 3E11FA47-71CA-11E1-9E33-C80AA9429562 | 39             | 39           |
| 3E11FA47-71CA-11E1-9E33-C80AA9429562 | 40             | 40           |
| 3E11FA47-71CA-11E1-9E33-C80AA9429562 | 41             | 41           |
| 3E11FA47-71CA-11E1-9E33-C80AA9429562 | 42             | 42           |
| 3E11FA47-71CA-11E1-9E33-C80AA9429562 | 43             | 43           |
...
```

为节省空间，MySQL 服务器可以定期压缩`mysql.gtid_executed`表，通过用跨越整个事务标识符间隔的单行替换每个这样的行集，如下所示：

```sql
+--------------------------------------+----------------+--------------+
| source_uuid                          | interval_start | interval_end |
|--------------------------------------+----------------+--------------|
| 3E11FA47-71CA-11E1-9E33-C80AA9429562 | 37             | 43           |
...
```

服务器可以使用名为`thread/sql/compress_gtid_table`的专用前台线程执行压缩。此线程不在`SHOW PROCESSLIST`的输出中列出，但可以在`threads`表中查看，如下所示：

```sql
mysql> SELECT * FROM performance_schema.threads WHERE NAME LIKE '%gtid%'\G
*************************** 1\. row ***************************
          THREAD_ID: 26
               NAME: thread/sql/compress_gtid_table
               TYPE: FOREGROUND
     PROCESSLIST_ID: 1
   PROCESSLIST_USER: NULL
   PROCESSLIST_HOST: NULL
     PROCESSLIST_DB: NULL
PROCESSLIST_COMMAND: Daemon
   PROCESSLIST_TIME: 1509
  PROCESSLIST_STATE: Suspending
   PROCESSLIST_INFO: NULL
   PARENT_THREAD_ID: 1
               ROLE: NULL
       INSTRUMENTED: YES
            HISTORY: YES
    CONNECTION_TYPE: NULL
       THREAD_OS_ID: 18677
```

当服务器启用二进制日志记录时，不使用此压缩方法，而是在每次二进制日志旋转时压缩`mysql.gtid_executed`表。但是，当服务器禁用二进制日志记录时，`thread/sql/compress_gtid_table`线程会休眠，直到执行了指定数量的事务，然后唤醒以压缩`mysql.gtid_executed`表。然后再休眠，直到发生相同数量的事务，然后再次唤醒以执行压缩，无限循环。在表被压缩之前经过的事务数量，因此压缩速率，由`gtid_executed_compression_period`系统变量的值控制。将该值设置为 0 意味着线程永远不会唤醒，这意味着不使用此显式压缩方法。相反，根据需要隐式进行压缩。

从 MySQL 8.0.17 开始，`InnoDB` 事务由一个独立进程写入到`mysql.gtid_executed`表，与非`InnoDB`事务分开。这个进程由不同的线程`innodb/clone_gtid_thread`控制。这个 GTID 持久化线程将 GTID 分组收集，将它们刷新到`mysql.gtid_executed`表，然后压缩表格。如果服务器同时有`InnoDB`事务和非`InnoDB`事务，它们分别写入到`mysql.gtid_executed`表，那么`compress_gtid_table`线程执行的压缩会干扰 GTID 持久化线程的工作，并且可能显著减慢速度。因此，从那个版本开始，建议将`gtid_executed_compression_period`设置为 0，这样`compress_gtid_table`线程就永远不会被激活。

从 MySQL 8.0.23 开始，`gtid_executed_compression_period`的默认值为 0，`InnoDB`和非`InnoDB`事务都由 GTID 持久化线程写入到`mysql.gtid_executed`表。

对于 MySQL 8.0.17 之前的版本，默认值为 1000 的`gtid_executed_compression_period`可以使用，意味着每 1000 个事务后对表进行压缩，或者您可以选择另一个值。在这些版本中，如果将值设置为 0 且禁用了二进制日志记录，则不会对`mysql.gtid_executed`表执行显式压缩，如果这样做，您应该准备好表可能需要的磁盘空间可能会大幅增加。

当启动服务器实例时，如果`gtid_executed_compression_period`设置为非零值并且启动了`thread/sql/compress_gtid_table`线程，在大多数服务器配置中，将为`mysql.gtid_executed`表执行显式压缩。在 MySQL 8.0.17 之前的版本中，启用二进制日志时，压缩是由于启动时二进制日志轮换触发的。在 MySQL 8.0.20 之后的版本中，压缩是由线程启动触发的。在这些版本之间的版本中，启动时不会进行压缩。
