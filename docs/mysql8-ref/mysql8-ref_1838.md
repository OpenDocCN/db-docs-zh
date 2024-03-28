# 25.7.12 NDB Cluster 复制冲突解决

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-conflict-resolution.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-conflict-resolution.html)

+   要求

+   源列控制

+   冲突解决控制

+   冲突解决函数

+   冲突解决异常表

+   冲突检测状态变量

+   示例

在涉及多个源（包括循环复制）的复制设置中，可能会出现不同源尝试使用不同数据更新副本上相同行的情况。NDB Cluster 复制中的冲突解决提供了一种通过允许使用用户定义的解决列来确定是否应在副本上应用给定源上的更新来解决此类冲突的方法。

NDB Cluster 支持的一些冲突解决类型（`NDB$OLD()`、`NDB$MAX()`和`NDB$MAX_DELETE_WIN()`；此外，在 NDB 8.0.30 及更高版本中，还有`NDB$MAX_INS()`和`NDB$MAX_DEL_WIN_INS()`）将此用户定义列实现为“时间戳”列（尽管其类型不能是`TIMESTAMP`，如本节后面所述）。这些类型的冲突解决总是基于逐行而不是基于事务的。基于时代的冲突解决函数`NDB$EPOCH()`和`NDB$EPOCH_TRANS()`比较了复制时代的顺序（因此这些函数是事务性的）。在冲突发生时，可以使用不同的方法来比较副本上的解决列值，如本节后面所述；所使用的方法可以设置为在单个表、数据库或服务器上操作，或者使用模式匹配在一个或多个表上操作。有关在`mysql.ndb_replication`表的`db`、`table_name`和`server_id`列中使用模式匹配的信息，请参见使用通配符进行匹配。

您还应该记住，确保解析列正确填充相关值是应用程序的责任，以便解析函数在确定是否应用更新时可以做出适当选择。

#### 要求

冲突解决的准备工作必须在源和副本上都进行。这些任务在以下列表中描述：

+   在写入二进制日志的源上，您必须确定要发送哪些列（所有列还是仅已更新的列）。这是通过在整个 MySQL Server 上应用**mysqld**启动选项`--ndb-log-updated-only`（稍后在本节中描述）来完成的，或者通过在`mysql.ndb_replication`表中放置适当的条目来在一个或多个特定表上完成（参见 ndb_replication Table）。

    注意

    如果您正在复制具有非常大列（如`TEXT`或`BLOB`列）的表，`--ndb-log-updated-only`也可以用于减小二进制日志的大小，并避免由于超过`max_allowed_packet`而导致的可能的复制失败。

    有关此问题的更多信息，请参见 Section 19.5.1.20, “Replication and max_allowed_packet”。

+   在副本上，您必须确定要应用哪种冲突解决方法（“最新时间戳获胜”，“相同时间戳获胜”，“主要获胜”，“主要获胜，完成事务”或无）。这是通过使用`mysql.ndb_replication`系统表来完成的，并适用于一个或多个特定表（参见 ndb_replication Table）。

+   NDB Cluster 还支持读冲突检测，即检测一个集群中对给定行的读取与另一个集群中对同一行的更新或删除之间的冲突。这需要通过在副本上将`ndb_log_exclusive_reads`设置为 1 来获得独占读锁。所有被冲突读取的行都将被记录在异常表中。有关更多信息，请参见读冲突检测和解决。

+   在 NDB 8.0.30 之前，`NDB`严格将`WRITE_ROW`事件应用为插入操作，要求不存在这样的行；也就是说，如果行已经存在，则传入的写操作总是被拒绝。（当使用除`NDB$MAX_INS()`或`NDB$MAX_DEL_WIN_INS()`之外的任何冲突解决函数时，仍然如此。）

    从 NDB 8.0.30 开始，当使用`NDB$MAX_INS()`或`NDB$MAX_DEL_WIN_INS()`时，`NDB`可以对`WRITE_ROW`事件进行幂等应用，将这样的事件映射到插入操作，当传入行不存在时，或者当传入行存在时映射到更新操作。

当使用函数`NDB$OLD()`，`NDB$MAX()`和`NDB$MAX_DELETE_WIN()`进行基于时间戳的冲突解决（以及从 NDB 8.0.30 开始使用`NDB$MAX_INS()`和`NDB$MAX_DEL_WIN_INS()`），我们通常将用于确定更新的列称为“时间戳”列。然而，此列的数据类型从不是`TIMESTAMP`；相反，其数据类型应为`INT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")（`INTEGER` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")）或`BIGINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")。 “时间戳”列还应为`UNSIGNED`和`NOT NULL`。

本节后面讨论的`NDB$EPOCH()`和`NDB$EPOCH_TRANS()`函数通过比较应用在主要和次要 NDB 集群上的复制时期的相对顺序来工作，并不使用时间戳。

#### 源列控制

我们可以根据“之前”和“之后”图像来看待更新操作——也就是说，在应用更新之前和之后的表状态。通常，当使用主键更新表时，“之前”图像并不是很重要；然而，当我们需要根据每次更新确定是否在副本上使用更新的值时，我们需要确保两个图像都写入源二进制日志。这是通过`--ndb-log-update-as-write`选项为**mysqld**完成的，稍后在本节中描述。

重要

决定是记录完整行还是仅更新列是在启动 MySQL 服务器时完成的，无法在线更改；您必须重新启动**mysqld**，或者使用不同的日志记录选项启动新的**mysqld**实例。

#### 冲突解决控制

冲突解决通常在可能发生冲突的服务器上启用。与日志记录方法选择一样，它是通过`mysql.ndb_replication`表中的条目启用的。

`NBT_UPDATED_ONLY_MINIMAL`和`NBT_UPDATED_FULL_MINIMAL`可与`NDB$EPOCH()`、`NDB$EPOCH2()`和`NDB$EPOCH_TRANS()`一起使用，因为这些不需要非主键列的“之前”值。需要旧值的冲突解决算法，如`NDB$MAX()`和`NDB$OLD()`，与这些`binlog_type`值不正确地配合。

#### 冲突解决函数

本节提供了关于可用于 NDB 复制中冲突检测和解决的函数的详细信息。

+   NDB$OLD()")

+   NDB$MAX()")

+   NDB$MAX_DELETE_WIN()")

+   NDB$MAX_INS()")

+   NDB$MAX_DEL_WIN_INS()")

+   NDB$EPOCH()")

+   NDB$EPOCH_TRANS()")

+   NDB$EPOCH2()")

+   NDB$EPOCH2_TRANS()")

##### NDB$OLD()

如果源数据和副本上的*`column_name`*的值相同，则应用更新；否则，在副本上不应用更新，并将异常写入日志。以下是伪代码示例：

```sql
if (*source_old_column_value* == *replica_current_column_value*)
  apply_update();
else
  log_exception();
```

此函数可用于“相同值获胜”冲突解决。这种冲突解决确保更新不会从错误的源应用于副本。

重要提示

此函数使用源数据的“之前”图像中的列值。

##### NDB$MAX()

对于更新或删除操作，如果源数据中给定行的“时间戳”列值高于副本中的值，则应用该操作；否则不应用于副本。以下是伪代码示例：

```sql
if (*source_new_column_value* > *replica_current_column_value*)
  apply_update();
```

此函数可用于“最大时间戳获胜”冲突解决。这种冲突解决确保在冲突发生时，最近更新的行版本是持久的版本。

除了拒绝具有与先前写操作相同主键的写操作外，此函数对写操作之间的冲突没有影响；仅当不存在使用相同主键的先前写操作时，才接受并应用该写操作。从 NDB 8.0.30 开始，您可以使用`NDB$MAX_INS()")`处理写操作之间的冲突解决。

重要

此函数使用源“after”图像的列值。

##### NDB$MAX_DELETE_WIN()

这是`NDB$MAX()`的变体。由于删除操作没有时间戳可用，因此使用`NDB$MAX()`进行的删除实际上被处理为`NDB$OLD`，但对于某些用例，这并不理想。对于`NDB$MAX_DELETE_WIN()`，如果来自源的添加或更新现有行的行的“时间戳”列值高于副本上的值，则应用该值。但是，删除操作始终被视为具有更高的值。如下伪代码所示：

```sql
if ( (*source_new_column_value* > *replica_current_column_value*)
        ||
      *operation.type* == "delete")
  apply_update();
```

此函数可用于“最大时间戳，删除获胜”冲突解决。这种冲突解决确保在冲突发生时，被删除或（其他方式）最近更新的行版本是持久的版本。

注意

与`NDB$MAX()`一样，此函数使用源“after”图像的列值。

##### NDB$MAX_INS()

此函数提供对冲突写操作的支持。这些冲突由“NDB$MAX_INS()”处理如下：

1.  如果没有冲突写操作，则应用此操作（与`NDB$MAX()`相同）。

1.  否则，应用“最大时间戳获胜”冲突解决，如下所示：

    1.  如果传入写操作的时间戳大于冲突写操作的时间戳，则应用传入操作。

    1.  如果传入写操作的时间戳*不*更大，则拒绝传入写操作。

处理插入操作时，`NDB$MAX_INS()`比较源和副本的时间戳，如下伪代码所示：

```sql
if (source_new_column_value > replica_current_column_value)
  apply_insert();
else
  log_exception();
```

对于更新操作，源的更新时间戳列值与副本的时间戳列值进行比较，如下所示：

```sql
if (source_new_column_value > replica_current_column_value)
  apply_update();
else
  log_exception();
```

这与`NDB$MAX()")`执行的操作相同。

对于删除操作，处理方式与`NDB$MAX()`（因此与`NDB$OLD()`）执行的操作相同，如下所示：

```sql
if (source_new_column_value == replica_current_column_value)
  apply_delete();
else
  log_exception();
```

`NDB$MAX_INS()`添加在 NDB 8.0.30 中。

##### NDB$MAX_DEL_WIN_INS()

此函数提供对冲突写操作的支持，以及类似于`NDB$MAX_DELETE_WIN()")`的“删除获胜”解决方案。写冲突由`NDB$MAX_DEL_WIN_INS()`处理如下：

1.  如果没有冲突的写入，应用这个（与 `NDB$MAX_DELETE_WIN()` 相同）。

1.  否则，应用“最大时间戳获胜”冲突解决，如下所示：

    1.  如果传入写入的时间戳大于冲突写入的时间戳，则应用传入操作。

    1.  如果传入写入的时间戳*不*更大，则拒绝传入的写入操作。

`NDB$MAX_DEL_WIN_INS()` 执行插入操作的处理可以用伪代码表示如下：

```sql
if (source_new_column_value > replica_current_column_value)
  apply_insert();
else
  log_exception();
```

对于更新操作，源的更新时间戳列值与副本的时间戳列值进行比较，如下所示（再次使用伪代码）：

```sql
if (source_new_column_value > replica_current_column_value)
  apply_update();
else
  log_exception();
```

删除使用“删除始终获胜”策略处理（与 `NDB$MAX_DELETE_WIN()` 相同）；`DELETE` 总是应用，而不考虑任何时间戳值，如下所示的伪代码所示：

```sql
if (operation.type == "delete")
  apply_delete();
```

对于更新和删除操作之间的冲突，此函数的行为与 `NDB$MAX_DELETE_WIN()` 完全相同。

`NDB$MAX_DEL_WIN_INS()` 添加在 NDB 8.0.30 中。

##### NDB$EPOCH()

`NDB$EPOCH()` 函数跟踪在副本集群上应用的复制时期的顺序，相对于在副本上发起的更改。这种相对顺序用于确定在副本上发起的更改是否与本地发起的任何更改同时发生，因此可能存在冲突。

在 `NDB$EPOCH()` 的描述中接下来的大部分内容也适用于 `NDB$EPOCH_TRANS()`。任何异常情况在文本中有注明。

`NDB$EPOCH()` 是不对称的，操作在一个 NDB 集群中的双向复制配置（有时称为“主动-主动”复制）。我们在这里将其操作的集群称为主集群，另一个称为从集群。主集群上的副本负责检测和处理冲突，而从集群上的副本不参与任何冲突检测或处理。

当主集群上的副本检测到冲突时，它会向自己的二进制日志中注入事件来补偿这些冲突；这确保了从集群最终重新与主集群对齐，从而使主集群和从集群保持一致。这种补偿和重新对齐机制要求主集群始终在与从集群的冲突中获胜，即主集群的更改始终优先于从集群的更改。这个“主始终获胜”的规则有以下含义：

+   一旦在主集群上提交更改数据的操作是完全持久的，并且不会被冲突检测和解决所撤消或回滚。

+   从主集群读取的数据是完全一致的。在主集群上提交的任何更改（本地或来自副本）后来都不会被撤销。

+   在从集群上更改数据的操作可能会在主集群确定它们存在冲突时稍后被撤销。

+   在辅助节点上读取的各行始终自洽，每行始终反映出辅助节点提交的状态或主节点提交的状态之一。

+   在辅助节点上读取的行集在给定的单个时间点上可能不一定一致。对于`NDB$EPOCH_TRANS()`来说，这是一个瞬时状态；对于`NDB$EPOCH()`来说，它可以是一个持久状态。

+   假设在足够长的时间段内没有任何冲突，辅助 NDB 集群上的所有数据（最终）都会与主节点的数据保持一致。

`NDB$EPOCH()`和`NDB$EPOCH_TRANS()`不需要任何用户模式修改或应用更改来提供冲突检测。然而，必须仔细考虑所使用的模式和访问模式，以验证整个系统是否在指定的限制内运行。

`NDB$EPOCH()`和`NDB$EPOCH_TRANS()`函数中的每一个都可以接受一个可选参数；这是用来表示时代低 32 位的比特数，应设置为不少于如下计算所示的值：

```sql
CEIL( LOG2( TimeBetweenGlobalCheckpoints / TimeBetweenEpochs ), 1)
```

对于这些配置参数的默认值（分别为 2000 和 100 毫秒），这给出了一个值为 5 位，因此默认值（6）应该足够，除非为`TimeBetweenGlobalCheckpoints`、`TimeBetweenEpochs`或两者都使用了其他值。值太小可能导致误报，而值太大可能导致数据库中浪费的空间过多。

`NDB$EPOCH()`和`NDB$EPOCH_TRANS()`都会将冲突行的条目插入相关的异常表中，前提是这些表已根据本节其他地方描述的相同异常表模式规则进行了定义（参见 NDB$OLD()")）。你必须在创建要使用的数据表之前创建任何异常表。

与本节讨论的其他冲突检测函数一样，通过在`mysql.ndb_replication`表中包含相关条目来激活`NDB$EPOCH()`和`NDB$EPOCH_TRANS()`（参见 ndb_replication 表）。在这种情况下，主 NDB 集群和辅助 NDB 集群的角色完全由`mysql.ndb_replication`表中的条目确定。

由于`NDB$EPOCH()`和`NDB$EPOCH_TRANS()`所采用的冲突检测算法是不对称的，你必须为主节点和辅助节点副本的`server_id`条目使用不同的值。

仅仅 DELETE 操作之间的冲突不足以触发使用`NDB$EPOCH()`或`NDB$EPOCH_TRANS()`的冲突，而且时代内的相对位置并不重要。

**NDB$EPOCH()的限制**

当使用`NDB$EPOCH()`执行冲突检测时，目前存在以下限制：

+   使用 NDB 集群时代边界来检测冲突，粒度与`TimeBetweenEpochs`成比例（默认值：100 毫秒）。最小冲突窗口是同时在两个集群上对同一数据进行并发更新时始终报告冲突的最短时间。这始终是一个非零长度的时间，并且大致与`2 *（延迟+排队+TimeBetweenEpochs）`成比例。这意味着——假设默认为`TimeBetweenEpochs`并忽略集群之间的任何延迟（以及任何排队延迟）——最小冲突窗口大小约为 200 毫秒。在查看预期应用程序“竞争”模式时，应考虑这个最小窗口。

+   使用`NDB$EPOCH()`和`NDB$EPOCH_TRANS()`函数的表需要额外的存储空间；每行需要额外的 1 到 32 位空间，具体取决于传递给函数的值。

+   删除操作之间的冲突可能导致主要和次要之间的分歧。当同时在两个集群上删除一行时，冲突可以被检测到，但不会被记录，因为行已被删除。这意味着在任何后续重新对齐操作的传播过程中不会检测到进一步的冲突，这可能导致分歧。

    删除应该外部串行化，或者路由到一个集群。或者，应该在这些删除和随后的任何插入事务中事务更新一个单独的行，以便跟踪冲突。这可能需要应用程序的更改。

+   当使用`NDB$EPOCH()`或`NDB$EPOCH_TRANS()`进行冲突检测时，目前仅支持双向“主动-主动”配置中的两个 NDB 集群。

+   具有`BLOB`或`TEXT`列的表目前不支持使用`NDB$EPOCH()`或`NDB$EPOCH_TRANS()`。

##### NDB$EPOCH_TRANS()

`NDB$EPOCH_TRANS()`扩展了`NDB$EPOCH()`函数。冲突使用“主要获胜”规则（参见 NDB$EPOCH()")）进行检测和处理，但额外条件是在发生冲突的同一事务中更新的任何其他行也被视为冲突。换句话说，`NDB$EPOCH()`在次要上重新对齐单个冲突行，而`NDB$EPOCH_TRANS()`在冲突事务上重新对齐。

此外，任何可检测依赖于冲突事务的事务也被视为冲突，这些依赖关系由次要集群的二进制日志内容确定。由于二进制日志仅包含数据修改操作（插入、更新和删除），因此只有重叠的数据修改用于确定事务之间的依赖关系。

`NDB$EPOCH_TRANS()`受到与`NDB$EPOCH()`相同的条件和限制，并且还要求所有事务 ID 都记录在次要的二进制日志中，使用`--ndb-log-transaction-id`设置为`ON`。这会增加可变的开销量（每行最多 13 个字节）。

废弃的`log_bin_use_v1_row_events`系统变量，默认值为`OFF`，*不*应该与`NDB$EPOCH_TRANS()`一起设置为`ON`。

参见 NDB$EPOCH()")。

##### NDB$EPOCH2()

`NDB$EPOCH2()`函数类似于`NDB$EPOCH()`，不同之处在于`NDB$EPOCH2()`提供了双向复制拓扑的删除-删除处理。在这种情况下，通过在每个源上设置`ndb_slave_conflict_role`系统变量的适当值（通常一个`PRIMARY`，一个`SECONDARY`）来为两个源分配主要和次要角色。完成此操作后，次要所做的修改会由主要反映回次要，然后有条件地应用这些修改。

##### NDB$EPOCH2_TRANS()

`NDB$EPOCH2_TRANS()`扩展了`NDB$EPOCH2()`函数。冲突的检测和处理方式相同，并为复制集群分配主要和次要角色，但额外条件是在发生冲突的同一事务中更新的任何其他行也被视为冲突。也就是说，`NDB$EPOCH2()`在次要上重新调整单个冲突行，而`NDB$EPOCH_TRANS()`重新调整冲突事务。

`NDB$EPOCH()`和`NDB$EPOCH_TRANS()`使用每行指定的元数据，每个最后修改的时代，在主键上确定来自辅助的复制行更改是否与本地提交的更改同时发生；同时发生的更改被视为冲突，随后的异常表更新和辅助的重新对齐。当主键上的行被删除时，不再有任何最后修改的时代可用来确定任何复制操作是否冲突，这意味着不会检测到冲突的删除操作。这可能导致分歧，例如一个集群上的删除与另一个集群上的删除和插入同时发生；这就是在使用`NDB$EPOCH()`和`NDB$EPOCH_TRANS()`时删除操作只能路由到一个集群的原因。

`NDB$EPOCH2()`通过忽略任何删除-删除冲突的问题—在主键上存储有关已删除行的信息—并避免任何潜在的结果分歧。通过将成功应用并从辅助复制的任何操作反射回辅助，可以在辅助上重新应用由来自主键的操作删除的操作。

当使用`NDB$EPOCH2()`时，您应该记住，辅助会应用从主键删除的操作，直到通过反射操作恢复新行。理论上，辅助上的后续插入或更新与主键上的删除冲突，但在这种情况下，我们选择忽略这一点，并允许辅助“获胜”，以防止集群之间的分歧。换句话说，在删除后，主键不会检测到冲突，而是立即采纳辅助的后续更改。因此，随着辅助向最终（稳定）状态前进，辅助的状态可以重新访问多个先前提交的状态，并且其中一些可能是可见的。

您还应该意识到，将所有操作从辅助反射回主键会增加主键的 logbinary 日志大小，以及对带宽、CPU 使用率和磁盘 I/O 的需求。

应用在辅助上的反射操作取决于辅助上目标行的状态。可以通过检查`Ndb_conflict_reflected_op_prepare_count`和`Ndb_conflict_reflected_op_discard_count`状态变量来跟踪在辅助上是否应用了反射更改。应用的更改数量简单地是这两个值之间的差异（请注意`Ndb_conflict_reflected_op_prepare_count`始终大于或等于`Ndb_conflict_reflected_op_discard_count`）。

事件仅在以下两个条件都为真时应用：

+   行的存在与否，即它是否存在，与事件类型一致。对于删除和更新操作，行必须已经存在。对于插入操作，行必须*不存在*。

+   行是由主要操作修改的。可能是通过执行反射操作完成修改的。

如果这两个条件不满足，则次要操作将被次要方丢弃。

#### 冲突解决异常表

要使用`NDB$OLD()`冲突解决函数，还需要为每个要使用此类冲突解决的`NDB`表创建一个对应的异常表。当使用`NDB$EPOCH()`或`NDB$EPOCH_TRANS()`时也是如此。这个表的名称与要应用冲突解决的表的名称相同，只是在末尾添加了字符串`$EX`。（例如，如果原始表的名称是`mytable`，则相应的异常表名称应为`mytable$EX`。）创建异常表的语法如下所示：

```sql
CREATE TABLE *original_table*$EX  (
    [NDB$]server_id INT UNSIGNED,
    [NDB$]source_server_id INT UNSIGNED,
    [NDB$]source_epoch BIGINT UNSIGNED,
    [NDB$]count INT UNSIGNED,

    [NDB$OP_TYPE ENUM('WRITE_ROW','UPDATE_ROW', 'DELETE_ROW',
      'REFRESH_ROW', 'READ_ROW') NOT NULL,]
    [NDB$CFT_CAUSE ENUM('ROW_DOES_NOT_EXIST', 'ROW_ALREADY_EXISTS',
      'DATA_IN_CONFLICT', 'TRANS_IN_CONFLICT') NOT NULL,]
    [NDB$ORIG_TRANSID BIGINT UNSIGNED NOT NULL,]

    *original_table_pk_columns*,

    [*orig_table_column*|*orig_table_column*$OLD|*orig_table_column*$NEW,]

    [*additional_columns*,]

    PRIMARY KEY([NDB$]server_id, [NDB$]source_server_id, [NDB$]source_epoch, [NDB$]count)
) ENGINE=NDB;
```

前四列是必需的。前四列的名称和与原始表主键列匹配的列的名称并不重要；但是，出于清晰和一致性的原因，我们建议您使用此处显示的`server_id`、`source_server_id`、`source_epoch`和`count`列的名称，并且对于与原始表主键匹配的列，使用与原始表中相同的名称。

如果异常表使用本节后面讨论的一个或多个可选列`NDB$OP_TYPE`、`NDB$CFT_CAUSE`或`NDB$ORIG_TRANSID`，则每个必需列也必须使用前缀`NDB$`命名。如果需要，即使您不定义任何可选列，也可以使用`NDB$`前缀命名必需列，但在这种情况下，所有四个必需列必须使用前缀命名。

在这些列之后，应按照用于定义原始表主键的顺序复制构成原始表主键的列。复制原始表主键列的列的数据类型应与原始列相同（或更大）。可以使用主键列的子集。

异常表必须使用`NDB`存储引擎。（本节后面将展示使用带有异常表的`NDB$OLD()`的示例。）

可以在复制的主键列后面选择性地定义其他列，但不能在它们之前；任何此类额外列都不能为 `NOT NULL`。NDB 集群支持三个额外的预定义可选列 `NDB$OP_TYPE`、`NDB$CFT_CAUSE` 和 `NDB$ORIG_TRANSID`，这些列在接下来的几段中描述。

`NDB$OP_TYPE`：此列可用于获取导致冲突的操作类型。如果使用此列，请按以下所示定义：

```sql
NDB$OP_TYPE ENUM('WRITE_ROW', 'UPDATE_ROW', 'DELETE_ROW',
    'REFRESH_ROW', 'READ_ROW') NOT NULL
```

`WRITE_ROW`、`UPDATE_ROW` 和 `DELETE_ROW` 操作类型代表用户发起的操作。`REFRESH_ROW` 操作是由冲突解决在补偿事务中生成的操作，从检测到冲突的集群发送回原始集群。`READ_ROW` 操作是由用户发起的读跟踪操作，使用独占行锁定义。

`NDB$CFT_CAUSE`：您可以定义一个可选列 `NDB$CFT_CAUSE`，提供注册冲突的原因。如果使用此列，则应按以下所示定义该列：

```sql
NDB$CFT_CAUSE ENUM('ROW_DOES_NOT_EXIST', 'ROW_ALREADY_EXISTS',
    'DATA_IN_CONFLICT', 'TRANS_IN_CONFLICT') NOT NULL
```

`ROW_DOES_NOT_EXIST` 可以报告为 `UPDATE_ROW` 和 `WRITE_ROW` 操作的原因；`ROW_ALREADY_EXISTS` 可以报告为 `WRITE_ROW` 事件的原因。当基于行的冲突函数检测到冲突时，报告 `DATA_IN_CONFLICT`；当事务冲突函数拒绝完整事务的所有操作时，报告 `TRANS_IN_CONFLICT`。

`NDB$ORIG_TRANSID`：如果使用 `NDB$ORIG_TRANSID` 列，则该列包含原始事务的 ID。应按以下方式定义此列：

```sql
NDB$ORIG_TRANSID BIGINT UNSIGNED NOT NULL
```

`NDB$ORIG_TRANSID` 是由 `NDB` 生成的 64 位值。此值可用于关联属于相同冲突事务的多个异常表条目，这些条目来自相同或不同的异常表。

附加的参考列，不是原始表的主键的一部分，可以命名为 `*`colname`*$OLD` 或 `*`colname`*$NEW`。 `*`colname`*$OLD` 引用更新和删除操作中的旧值，即包含 `DELETE_ROW` 事件的操作。 `*`colname`*$NEW` 可用于引用插入和更新操作中的新值，换句话说，使用 `WRITE_ROW` 事件、`UPDATE_ROW` 事件或两种事件的操作。如果冲突操作未为给定的非主键参考列提供值，则异常表行包含 `NULL` 或该列的定义默认值。

重要

当数据表设置为复制时，将读取 `mysql.ndb_replication` 表，因此在创建要复制的表之前，必须将对应于要复制的表的行插入到 `mysql.ndb_replication` 中。

#### 冲突检测状态变量

几个状态变量可用于监视冲突检测。您可以通过`Ndb_conflict_fn_epoch`系统状态变量的当前值，查看自此副本上次从头启动以来，由`NDB$EPOCH()`发现的冲突行数。

`Ndb_conflict_fn_epoch_trans`提供了由`NDB$EPOCH_TRANS()`直接发现的冲突行数。`Ndb_conflict_fn_epoch2`和`Ndb_conflict_fn_epoch2_trans`分别显示了由`NDB$EPOCH2()`和`NDB$EPOCH2_TRANS()`发现的冲突行数。实际重新对齐的行数，包括由于它们属于或依赖于与其他冲突行相同事务的行受到影响的行数，由`Ndb_conflict_trans_row_reject_count`给出。

另一个服务器状态变量`Ndb_conflict_fn_max`提供了自上次**mysqld**启动以来，由于“最大时间戳获胜”冲突解决而未在当前 SQL 节点上应用的行数。`Ndb_conflict_fn_max_del_win`提供了基于`NDB$MAX_DELETE_WIN()`结果的冲突解决已应用的次数。

NDB 8.0.30 及更高版本提供了`Ndb_conflict_fn_max_ins`，用于跟踪“更大时间戳获胜”处理已应用于写操作的次数（使用`NDB$MAX_INS()`）；由状态变量`Ndb_conflict_fn_max_del_win_ins`提供了“相同时间戳获胜”写操作处理已应用的次数（由`NDB$MAX_DEL_WIN_INS()`实现）。

自上次**mysqld**重新启动以来，由于“相同时间戳获胜”冲突解决而未应用的行数由全局状态变量`Ndb_conflict_fn_old`给出。除了递增`Ndb_conflict_fn_old`外，未使用的行的主键被插入到异常表中，如本节其他地方所述。

另请参阅 Section 25.4.3.9.3, “NDB Cluster Status Variables”。

#### 示例

以下示例假定您已经有一个正常工作的 NDB Cluster 复制设置，如 Section 25.7.5, “Preparing the NDB Cluster for Replication” 和 Section 25.7.6, “Starting NDB Cluster Replication (Single Replication Channel)”") 中所述。

**NDB$MAX() 示例。** 假设你希望在表 `test.t1` 上启用“最大时间戳获胜”冲突解决，使用列 `mycol` 作为“时间戳”。可以通过以下步骤完成：

1.  确保你已经使用 `--ndb-log-update-as-write=OFF` 启动了源 **mysqld**。

1.  在源上执行这个 `INSERT` 语句：

    ```sql
    INSERT INTO mysql.ndb_replication
        VALUES ('test', 't1', 0, NULL, 'NDB$MAX(mycol)');
    ```

    注意

    如果 `ndb_replication` 表尚不存在，则必须创建它。请参阅 ndb_replication Table。

    将 0 插入到 `server_id` 列中表示所有访问该表的 SQL 节点应该使用冲突解决。如果你只想在特定的 **mysqld** 上使用冲突解决，使用实际的服务器 ID。

    将 `NULL` 插入到 `binlog_type` 列中与插入 0 (`NBT_DEFAULT`) 具有相同的效果；使用服务器默认值。

1.  创建 `test.t1` 表：

    ```sql
    CREATE TABLE test.t1 (
        *columns*
        mycol INT UNSIGNED,
        *columns*
    ) ENGINE=NDB;
    ```

    现在，当对该表执行更新时，将应用冲突解决，并将具有 `mycol` 最大值的行版本写入副本。

注意

其他 `binlog_type` 选项，如 `NBT_UPDATED_ONLY_USE_UPDATE` (`6`)，应该使用 `ndb_replication` 表来控制源上的日志记录，而不是使用命令行选项。

**NDB$OLD() 示例。** 假设正在复制一个 `NDB` 表，如此处定义的表，并且你希望为更新到该表的“相同时间戳获胜”冲突解决启用：

```sql
CREATE TABLE test.t2  (
    a INT UNSIGNED NOT NULL,
    b CHAR(25) NOT NULL,
    *columns*,
    mycol INT UNSIGNED NOT NULL,
    *columns*,
    PRIMARY KEY pk (a, b)
)   ENGINE=NDB;
```

需要按照以下顺序执行以下步骤：

1.  首先——*在*创建 `test.t2` 之前——你必须像这样向 `mysql.ndb_replication` 表中插入一行：

    ```sql
    INSERT INTO mysql.ndb_replication
        VALUES ('test', 't2', 0, 0, 'NDB$OLD(mycol)');
    ```

    `binlog_type` 列的可能值在本节中已经显示；在这种情况下，我们使用 `0` 来指定使用服务器默认的日志记录行为。值 `'NDB$OLD(mycol)'` 应该插入到 `conflict_fn` 列中。

1.  为`test.t2`创建一个适当的异常表。此处显示的表创建语句包括所有必需列；任何额外列必须在这些列之后声明，并在表的主键定义之前。

    ```sql
    CREATE TABLE test.t2$EX  (
        server_id INT UNSIGNED,
        source_server_id INT UNSIGNED,
        source_epoch BIGINT UNSIGNED,
        count INT UNSIGNED,
        a INT UNSIGNED NOT NULL,
        b CHAR(25) NOT NULL,

        [*additional_columns*,]

        PRIMARY KEY(server_id, source_server_id, source_epoch, count)
    )   ENGINE=NDB;
    ```

    我们可以为给定冲突的类型、原因和起始事务 ID 包含额外列。我们也不需要为原始表中的所有主键列提供匹配列。这意味着您可以像这样创建异常表：

    ```sql
    CREATE TABLE test.t2$EX  (
        NDB$server_id INT UNSIGNED,
        NDB$source_server_id INT UNSIGNED,
        NDB$source_epoch BIGINT UNSIGNED,
        NDB$count INT UNSIGNED,
        a INT UNSIGNED NOT NULL,

        NDB$OP_TYPE ENUM('WRITE_ROW','UPDATE_ROW', 'DELETE_ROW',
          'REFRESH_ROW', 'READ_ROW') NOT NULL,
        NDB$CFT_CAUSE ENUM('ROW_DOES_NOT_EXIST', 'ROW_ALREADY_EXISTS',
          'DATA_IN_CONFLICT', 'TRANS_IN_CONFLICT') NOT NULL,
        NDB$ORIG_TRANSID BIGINT UNSIGNED NOT NULL,

        [*additional_columns*,]

        PRIMARY KEY(NDB$server_id, NDB$source_server_id, NDB$source_epoch, NDB$count)
    )   ENGINE=NDB;
    ```

    注意

    由于我们在表定义中至少包含了`NDB$OP_TYPE`、`NDB$CFT_CAUSE`或`NDB$ORIG_TRANSID`中的一个列，因此四个必需列需要使用`NDB$`前缀。

1.  如前所示创建`test.t2`表。

这些步骤必须针对每个要使用`NDB$OLD()`执行冲突解决的表进行遵循。对于每个这样的表，必须在`mysql.ndb_replication`中有一个相应的行，并且在被复制的表所在的同一数据库中必须有一个异常表。

**阅读冲突检测和解决。** NDB Cluster 还支持跟踪读操作，这使得在循环复制设置中可以管理一个集群中给定行的读取与另一个集群中相同行的更新或删除之间的冲突。此示例使用`employee`和`department`表来模拟这样一种情况：在源集群（以下简称为集群*A*）中将员工从一个部门移动到另一个部门，而副本集群（以下简称为*B*）在交错事务中更新员工以前部门的员工计数。

数据表已使用以下 SQL 语句创建：

```sql
# Employee table
CREATE TABLE employee (
    id INT PRIMARY KEY,
    name VARCHAR(2000),
    dept INT NOT NULL
)   ENGINE=NDB;

# Department table
CREATE TABLE department (
    id INT PRIMARY KEY,
    name VARCHAR(2000),
    members INT
)   ENGINE=NDB;
```

两个表的内容包括以下`SELECT`语句的（部分）输出中显示的行：

```sql
mysql> SELECT id, name, dept FROM employee;
+---------------+------+
| id   | name   | dept |
+------+--------+------+
...
| 998  |  Mike  | 3    |
| 999  |  Joe   | 3    |
| 1000 |  Mary  | 3    |
...
+------+--------+------+

mysql> SELECT id, name, members FROM department;
+-----+-------------+---------+
| id  | name        | members |
+-----+-------------+---------+
...
| 3   | Old project | 24      |
...
+-----+-------------+---------+
```

我们假设我们已经在使用包含四个必需列（并且这些列用于此表的主键）的异常表，操作类型和原因的可选列，以及原始表的主键列，使用此处显示的 SQL 语句创建：

```sql
CREATE TABLE employee$EX  (
    NDB$server_id INT UNSIGNED,
    NDB$source_server_id INT UNSIGNED,
    NDB$source_epoch BIGINT UNSIGNED,
    NDB$count INT UNSIGNED,

    NDB$OP_TYPE ENUM( 'WRITE_ROW','UPDATE_ROW', 'DELETE_ROW',
                      'REFRESH_ROW','READ_ROW') NOT NULL,
    NDB$CFT_CAUSE ENUM( 'ROW_DOES_NOT_EXIST',
                        'ROW_ALREADY_EXISTS',
                        'DATA_IN_CONFLICT',
                        'TRANS_IN_CONFLICT') NOT NULL,

    id INT NOT NULL,

    PRIMARY KEY(NDB$server_id, NDB$source_server_id, NDB$source_epoch, NDB$count)
)   ENGINE=NDB;
```

假设在两个集群上发生了两个同时事务。在集群*A*上，我们创建一个新部门，然后将员工编号 999 移入该部门，使用以下 SQL 语句：

```sql
BEGIN;
  INSERT INTO department VALUES (4, "New project", 1);
  *UPDATE employee SET dept = 4 WHERE id = 999;* COMMIT;
```

与此同时，在集群*B*上，另一个事务从`employee`中读取，如下所示：

```sql
BEGIN;
  *SELECT name FROM employee WHERE id = 999;* UPDATE department SET members = members - 1  WHERE id = 3;
commit;
```

冲突解决机制通常不会检测到冲突的事务，因为冲突是在读取（`SELECT`）和更新操作之间。您可以通过在复制集群上执行`SET` `ndb_log_exclusive_reads` `= 1`来解决此问题。以这种方式获取独占读锁会导致在源上读取的任何行在复制集群上被标记为需要冲突解决。如果在记录这些事务之前以这种方式启用独占读取，则在集群*B*上的读取将被跟踪并发送到集群*A*进行解决；随后检测到员工行上的冲突，并且在集群*B*上的事务被中止。

冲突在异常表（在集群*A*上）中注册为`READ_ROW`操作（参见冲突解决异常表，了解操作类型的描述），如下所示：

```sql
mysql> SELECT id, NDB$OP_TYPE, NDB$CFT_CAUSE FROM employee$EX;
+-------+-------------+-------------------+
| id    | NDB$OP_TYPE | NDB$CFT_CAUSE     |
+-------+-------------+-------------------+
...
| 999   | READ_ROW    | TRANS_IN_CONFLICT |
+-------+-------------+-------------------+
```

在读操作中找到的任何现有行都会被标记。这意味着由于同一冲突导致的多行可能会在异常表中记录，如通过检查在同时事务中在集群*A*上进行更新和在集群*B*上从相同表读取多行之间的冲突的影响所示。在集群*A*上执行的事务如下所示：

```sql
BEGIN;
  INSERT INTO department VALUES (4, "New project", 0);
  *UPDATE employee SET dept = 4 WHERE dept = 3;* SELECT COUNT(*) INTO @count FROM employee WHERE dept = 4;
  UPDATE department SET members = @count WHERE id = 4;
COMMIT;
```

同时，在集群*B*上运行包含以下语句的事务：

```sql
SET ndb_log_exclusive_reads = 1;  # Must be set if not already enabled
...
BEGIN;
  *SELECT COUNT(*) INTO @count FROM employee WHERE dept = 3 FOR UPDATE;* UPDATE department SET members = @count WHERE id = 3;
COMMIT;
```

在这种情况下，第二个事务的`SELECT`中匹配`WHERE`条件的所有三行都被读取，并因此在异常表中标记，如下所示：

```sql
mysql> SELECT id, NDB$OP_TYPE, NDB$CFT_CAUSE FROM employee$EX;
+-------+-------------+-------------------+
| id    | NDB$OP_TYPE | NDB$CFT_CAUSE     |
+-------+-------------+-------------------+
...
| 998   | READ_ROW    | TRANS_IN_CONFLICT |
| 999   | READ_ROW    | TRANS_IN_CONFLICT |
| 1000  | READ_ROW    | TRANS_IN_CONFLICT |
...
+-------+-------------+-------------------+
```

读跟踪仅基于现有行执行。基于给定条件的读取仅跟踪*找到*的任何行的冲突，而不是插入在交错事务中的任何行。这类似于在单个 NDB 集群实例中执行独占行锁定的方式。

**插入冲突检测和解决示例（NDB 8.0.30 及更高版本）。** 以下示例说明了在 NDB 8.0.30 中添加的插入冲突检测功能的使用。我们假设我们正在复制数据库`test`中的两个表`t1`和`t2`，并且希望对`t1`使用`NDB$MAX_INS()`进行插入冲突检测，对`t2`使用`NDB$MAX_DEL_WIN_INS()`进行插入冲突检测。这两个数据表直到设置过程的后期才会创建。

设置插入冲突解决类似于设置其他冲突检测和解决算法，如前面的示例所示。如果用于配置二进制日志记录和冲突解决的`mysql.ndb_replication`表尚不存在，则首先需要创建它，如下所示：

```sql
CREATE TABLE mysql.ndb_replication (
    db VARBINARY(63),
    table_name VARBINARY(63),
    server_id INT UNSIGNED,
    binlog_type INT UNSIGNED,
    conflict_fn VARBINARY(128),
    PRIMARY KEY USING HASH (db, table_name, server_id)
) ENGINE=NDB 
PARTITION BY KEY(db,table_name);
```

`ndb_replication`表是基于每个表的基础操作的；也就是说，我们需要插入一行包含表信息、`binlog_type`值、要使用的冲突解决函数以及时间戳列（`X`）的名称，就像这样：

```sql
INSERT INTO mysql.ndb_replication VALUES ("test", "t1", 0, 7, "NDB$MAX_INS(X)");
INSERT INTO mysql.ndb_replication VALUES ("test", "t2", 0, 7, "NDB$MAX_DEL_WIN_INS(X)");
```

在这里，我们将 binlog_type 设置为`NBT_FULL_USE_UPDATE`（`7`），这意味着始终记录完整行。有关其他可能值，请参见 ndb_replication Table。

您还可以为每个需要使用冲突解决的`NDB`表创建一个异常表。异常表记录由给定表的冲突解决函数拒绝的所有行。可以使用以下两个 SQL 语句为表`t1`和`t2`创建用于复制冲突检测的异常表：

```sql
CREATE TABLE `t1$EX` (
    NDB$server_id INT UNSIGNED,
    NDB$master_server_id INT UNSIGNED,
    NDB$master_epoch BIGINT UNSIGNED,
    NDB$count INT UNSIGNED,
    NDB$OP_TYPE ENUM('WRITE_ROW', 'UPDATE_ROW', 'DELETE_ROW', 
                     'REFRESH_ROW', 'READ_ROW') NOT NULL,
    NDB$CFT_CAUSE ENUM('ROW_DOES_NOT_EXIST', 'ROW_ALREADY_EXISTS',
                       'DATA_IN_CONFLICT', 'TRANS_IN_CONFLICT') NOT NULL,
    a INT NOT NULL,
    PRIMARY KEY(NDB$server_id, NDB$master_server_id, 
                NDB$master_epoch, NDB$count)
) ENGINE=NDB;

CREATE TABLE `t2$EX` (
    NDB$server_id INT UNSIGNED,
    NDB$master_server_id INT UNSIGNED,
    NDB$master_epoch BIGINT UNSIGNED,
    NDB$count INT UNSIGNED,
    NDB$OP_TYPE ENUM('WRITE_ROW', 'UPDATE_ROW', 'DELETE_ROW',
                     'REFRESH_ROW', 'READ_ROW') NOT NULL,
    NDB$CFT_CAUSE ENUM( 'ROW_DOES_NOT_EXIST', 'ROW_ALREADY_EXISTS',
                        'DATA_IN_CONFLICT', 'TRANS_IN_CONFLICT') NOT NULL,
    a INT NOT NULL,
    PRIMARY KEY(NDB$server_id, NDB$master_server_id, 
                NDB$master_epoch, NDB$count)
) ENGINE=NDB;
```

最后，在刚刚显示的创建异常表之后，您可以创建要复制并受冲突解决控制的数据表，使用以下两个 SQL 语句：

```sql
CREATE TABLE t1 (
    a INT PRIMARY KEY, 
    b VARCHAR(32), 
    X INT UNSIGNED
) ENGINE=NDB;

CREATE TABLE t2 (
    a INT PRIMARY KEY, 
    b VARCHAR(32), 
    X INT UNSIGNED
) ENGINE=NDB;
```

对于每个表，`X`列被用作时间戳列。

一旦在源上创建了`t1`和`t2`，它们就会被复制并假定存在于源和副本上。在本示例的其余部分中，我们使用`mysqlS>`表示连接到源的**mysql**客户端，使用`mysqlR>`表示在副本上运行的**mysql**客户端。

首先，我们在源上分别插入一行到每个表中，就像这样：

```sql
mysqlS> INSERT INTO t1 VALUES (1, 'Initial X=1', 1);
Query OK, 1 row affected (0.01 sec)

mysqlS> INSERT INTO t2 VALUES (1, 'Initial X=1', 1);
Query OK, 1 row affected (0.01 sec)
```

我们可以确定这两行在不引起任何冲突的情况下被复制，因为在在源上发出`INSERT`语句之前，副本上的表不包含任何行。我们可以通过从副本中的表中选择来验证这一点，就像这样：

```sql
mysqlR> TABLE t1 ORDER BY a;
+---+-------------+------+
| a | b           | X    |
+---+-------------+------+
| 1 | Initial X=1 |    1 |
+---+-------------+------+
1 row in set (0.00 sec)

mysqlR> TABLE t2 ORDER BY a;
+---+-------------+------+
| a | b           | X    |
+---+-------------+------+
| 1 | Initial X=1 |    1 |
+---+-------------+------+
1 row in set (0.00 sec)
```

接下来，我们在副本中插入新行到表中，就像这样：

```sql
mysqlR> INSERT INTO t1 VALUES (2, 'Replica X=2', 2);
Query OK, 1 row affected (0.01 sec)

mysqlR> INSERT INTO t2 VALUES (2, 'Replica X=2', 2);
Query OK, 1 row affected (0.01 sec)
```

现在我们在源上插入具有更大时间戳（`X`）列值的冲突行，使用下面显示的语句：

```sql
mysqlS> INSERT INTO t1 VALUES (2, 'Source X=20', 20);
Query OK, 1 row affected (0.01 sec)

mysqlS> INSERT INTO t2 VALUES (2, 'Source X=20', 20);
Query OK, 1 row affected (0.01 sec)
```

现在我们通过再次从副本上的两个表中选择来观察结果，就像这样：

```sql
mysqlR> TABLE t1 ORDER BY a;
+---+-------------+-------+
| a | b           | X     |
+---+-------------+-------+
| 1 | Initial X=1 |    1  |
+---+-------------+-------+
| 2 | Source X=20 |   20  |
+---+-------------+-------+
2 rows in set (0.00 sec)

mysqlR> TABLE t2 ORDER BY a;
+---+-------------+-------+
| a | b           | X     |
+---+-------------+-------+
| 1 | Initial X=1 |    1  |
+---+-------------+-------+
| 1 | Source X=20 |   20  |
+---+-------------+-------+
2 rows in set (0.00 sec)
```

在源上插入的行，其时间戳大于副本中冲突行的时间戳，已替换了那些行。在副本上，我们接下来插入两行新行，这些行不与`t1`或`t2`中的任何现有行冲突，就像这样：

```sql
mysqlR> INSERT INTO t1 VALUES (3, 'Slave X=30', 30);
Query OK, 1 row affected (0.01 sec)

mysqlR> INSERT INTO t2 VALUES (3, 'Slave X=30', 30);
Query OK, 1 row affected (0.01 sec)
```

在源上插入具有相同主键值（`3`）的更多行会引起冲突，但这次我们使用时间戳列中小于副本中冲突行中相同列中的时间戳的值。

```sql
mysqlS> INSERT INTO t1 VALUES (3, 'Source X=3', 3);
Query OK, 1 row affected (0.01 sec)

mysqlS> INSERT INTO t2 VALUES (3, 'Source X=3', 3);
Query OK, 1 row affected (0.01 sec)
```

通过查询表格，我们可以看到源端插入的两行都被副本端拒绝了，并且副本端之前插入的行也没有被覆盖，就像在副本端的**mysql**客户端中所展示的那样：

```sql
mysqlR> TABLE t1 ORDER BY a;
+---+--------------+-------+
| a | b            | X     |
+---+--------------+-------+
| 1 |  Initial X=1 |    1  |
+---+--------------+-------+
| 2 |  Source X=20 |   20  |
+---+--------------+-------+
| 3 | Replica X=30 |   30  |
+---+--------------+-------+
3 rows in set (0.00 sec)

mysqlR> TABLE t2 ORDER BY a;
+---+--------------+-------+
| a | b            | X     |
+---+--------------+-------+
| 1 |  Initial X=1 |    1  |
+---+--------------+-------+
| 2 |  Source X=20 |   20  |
+---+--------------+-------+
| 3 | Replica X=30 |   30  |
+---+--------------+-------+
3 rows in set (0.00 sec)
```

您可以在异常表中查看被拒绝的行的信息，如下所示：

```sql
mysqlR> SELECT  NDB$server_id, NDB$master_server_id, NDB$count,
      >         NDB$OP_TYPE, NDB$CFT_CAUSE, a
      > FROM t1$EX
      > ORDER BY NDB$count\G
*************************** 1\. row ***************************
NDB$server_id       : 2
NDB$master_server_id: 1
NDB$count           : 1
NDB$OP_TYPE         : WRITE_ROW
NDB$CFT_CAUSE       : DATA_IN_CONFLICT
a                   : 3 1 row in set (0.00 sec)

mysqlR> SELECT  NDB$server_id, NDB$master_server_id, NDB$count,
      >         NDB$OP_TYPE, NDB$CFT_CAUSE, a
      > FROM t2$EX
      > ORDER BY NDB$count\G
*************************** 1\. row ***************************
NDB$server_id       : 2
NDB$master_server_id: 1
NDB$count           : 1
NDB$OP_TYPE         : WRITE_ROW
NDB$CFT_CAUSE       : DATA_IN_CONFLICT
a                   : 3 1 row in set (0.00 sec)
```

正如我们之前所看到的，源端插入的其他行并没有被副本端拒绝，只有那些时间戳值较小于副本端冲突行的行被拒绝。
