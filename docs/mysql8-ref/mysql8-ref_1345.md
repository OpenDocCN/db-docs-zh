> 译文：[`dev.mysql.com/doc/refman/8.0/en/replication-gtids-lifecycle.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-gtids-lifecycle.html)

#### 19.1.3.2 GTID 生命周期

GTID 的生命周期包括以下步骤：

1.  事务在源端执行并提交。此客户端事务被分配一个由源的 UUID 和尚未在此服务器上使用的最小非零事务序列号组成的 GTID。GTID 被写入源的二进制日志（在日志中的事务之前立即写入）。如果客户端事务未写入二进制日志（例如，因为事务被过滤掉，或者事务是只读的），则不会分配 GTID。

1.  如果为事务分配了 GTID，则在提交时通过将其写入二进制日志的方式原子地持久化该 GTID（作为 `Gtid_log_event`）。每当二进制日志被轮换或服务器关闭时，服务器将为写入到上一个二进制日志文件中的所有事务写入 GTID 到 `mysql.gtid_executed` 表中。

1.  如果为事务分配了 GTID，则在事务提交后不久通过将其添加到 `gtid_executed` 系统变量 (`@@GLOBAL.gtid_executed`) 中的 GTID 集合来非原子化地外部化该 GTID。此 GTID 集合包含所有已提交 GTID 事务的表示，并且在复制中用作表示服务器状态的令牌。启用二进制日志记录（源端所需）后，`gtid_executed` 系统变量中的 GTID 集合是应用的所有事务的完整记录，但 `mysql.gtid_executed` 表不是，因为最近的历史仍在当前二进制日志文件中。

1.  在二进制日志数据传输到副本并存储在副本的中继日志中（使用已建立的机制进行此过程，详见第 19.2 节，“复制实现”），副本读取 GTID 并将其设置为其 `gtid_next` 系统变量的值。这告诉副本下一个事务必须使用此 GTID 记录。重要的是要注意，副本在会话上下文中设置 `gtid_next`。

1.  副本验证尚未有任何线程拥有`gtid_next`中的 GTID 以处理事务。通过首先读取和检查复制的事务的 GTID，然后再处理事务本身，副本确保不仅在副本上未应用具有此 GTID 的先前事务，而且也没有其他会话已经读取此 GTID 但尚未提交关联事务。因此，如果多个客户端尝试并发应用相同的事务，服务器通过只允许其中一个执行来解决此问题。副本的`gtid_owned`系统变量（`@@GLOBAL.gtid_owned`）显示当前正在使用的每个 GTID 及拥有它的线程的 ID。如果 GTID 已经被使用，不会引发错误，并且自动跳过功能用于忽略该事务。

1.  如果 GTID 尚未被使用，副本将应用复制的事务。因为`gtid_next`已设置为源已分配的 GTID，副本不会尝试为此事务生成新的 GTID，而是使用存储在`gtid_next`中的 GTID。

1.  如果副本上启用了二进制日志记录，GTID 将在提交时以原子方式持久化，通过在事务开始时将其写入二进制日志（作为`Gtid_log_event`）。每当二进制日志轮换或服务器关闭时，服务器将为之前写入的所有事务写入的 GTID 写入`mysql.gtid_executed`表。

1.  如果副本上禁用了二进制日志记录，则通过直接将其写入`mysql.gtid_executed`表来以原子方式持久化 GTID。MySQL 会向事务追加一个语句，将 GTID 插入表中。从 MySQL 8.0 开始，对于 DDL 语句以及 DML 语句，此操作都是原子的。在这种情况下，`mysql.gtid_executed`表是副本上应用的事务的完整记录。

1.  在副本上提交的事务后不久，GTID 将通过将其添加到副本的`gtid_executed`系统变量（`@@GLOBAL.gtid_executed`）中的 GTID 集合中非原子化地外部化。对于源，此 GTID 集合包含所有已提交 GTID 事务集合的表示。如果副本上禁用了二进制日志记录，则`mysql.gtid_executed`表也是副本上应用的事务的完整记录。如果副本上启用了二进制日志记录，意味着某些 GTID 仅记录在二进制日志中，则`gtid_executed`系统变量中的 GTID 集合是唯一的完整记录。

在源端完全被过滤掉的客户端事务不会被分配 GTID，因此它们不会被添加到`gtid_executed`系统变量的事务集合中，也不会被添加到`mysql.gtid_executed`表中。然而，在副本上完全被过滤掉的复制事务的 GTID 会被保留。如果在副本上启用了二进制日志记录，被过滤掉的事务会被写入二进制日志作为一个`Gtid_log_event`，然后是一个只包含`BEGIN`和`COMMIT`语句的空事务。如果禁用了二进制日志记录，被过滤掉的事务的 GTID 会被写入`mysql.gtid_executed`表中。保留被过滤掉事务的 GTID 确保了`mysql.gtid_executed`表和`gtid_executed`系统变量中的 GTID 集合可以被压缩。它还确保了如果副本重新连接到源端，被过滤掉的事务不会再次被检索，如第 19.1.3.3 节“GTID 自动定位”中所解释的那样。

在多线程副本（具有`replica_parallel_workers > 0`或`slave_parallel_workers > 0`）上，事务可以并行应用，因此复制事务可以无序提交（除非设置了`replica_preserve_commit_order=1`或`slave_preserve_commit_order=1`）。当发生这种情况时，`gtid_executed`系统变量中的 GTID 集合包含多个 GTID 范围之间的间隙。（在源端或单线程副本上，GTID 是单调递增的，数字之间没有间隙。）多线程副本上的间隙仅出现在最近应用的事务之间，并且随着复制的进行而填补。当使用`STOP REPLICA`语句干净地停止复制线程时，正在进行的事务会被应用，以便填补间隙。在发生诸如服务器故障或使用`KILL`语句停止复制线程等关闭事件时，这些间隙可能会保留。

##### 哪些更改会被分配一个 GTID？

典型情况是服务器为已提交的事务生成一个新的 GTID。然而，除了事务之外，GTIDs 也可以分配给其他更改，并且在某些情况下，单个事务可以被分配多个 GTIDs。

每个写入二进制日志的数据库更改（DDL 或 DML）都被分配一个 GTID。这包括自动提交的更改，以及使用`BEGIN`和`COMMIT`或`START TRANSACTION`语句提交的更改。GTID 也分配给数据库的创建、修改或删除，以及非表数据库对象（如存储过程、函数、触发器、事件、视图、用户、角色或授权）。

非事务更新以及事务更新都被分配 GTID。此外，对于非事务更新，如果在尝试写入二进制日志缓存时发生磁盘写入失败，从而在二进制日志中创建了一个间隙，那么生成的事件日志事件将被分配一个 GTID。

当一个表被二进制日志中的生成语句自动删除时，该语句被分配一个 GTID。当一个复制开始应用来自刚刚启动的源的事件时，临时表会被自动删除，并且当使用基于语句的复制（`binlog_format=STATEMENT`）并且一个具有打开临时表的用户会话断开连接时。使用`MEMORY`存储引擎的表在服务器启动后第一次访问时会自动删除，因为在关闭期间可能会丢失行。

当一个事务在原始服务器上没有写入二进制日志时，服务器不会为其分配 GTID。这包括被回滚的事务和在原始服务器上禁用二进制日志记录时执行的事务，无论是全局禁用（在服务器配置中指定`--skip-log-bin`）还是对会话禁用（`SET @@SESSION.sql_log_bin = 0`）。当使用基于行的复制时（`binlog_format=ROW`），这也包括无操作事务。

XA 事务为事务的`XA PREPARE`阶段和`XA COMMIT`或`XA ROLLBACK`阶段分配单独的 GTID。XA 事务被持久准备，以便用户在失败的情况下（在复制拓扑中可能包括故障转移到另一台服务器）提交或回滚它们。因此，事务的两个部分被分别复制，因此它们必须有自己的 GTID，即使一个被回滚的非 XA 事务不会有 GTID。

在以下特殊情况下，单个语句可以生成多个事务，因此被分配多个 GTID：

+   调用了提交多个事务的存储过程。为存储过程提交的每个事务生成一个 GTID。

+   一个多表`DROP TABLE`语句会删除不同类型的表。如果任何表使用不支持原子 DDL 的存储引擎，或者任何表是临时表，可能会生成多个 GTID。

+   当使用基于行的复制（`binlog_format=ROW`）时，会发出`CREATE TABLE ... SELECT`语句。为`CREATE TABLE`操作生成一个 GTID，并为行插入操作生成一个 GTID。

##### `gtid_next`系统变量

默认情况下，在用户会话中提交的新事务，服务器会自动生成并分配新的 GTID。在副本上应用事务时，会保留来自原始服务器的 GTID。您可以通过设置`gtid_next`系统变量的会话值来更改此行为：

+   当`gtid_next`设置为`AUTOMATIC`时，这是默认值，事务提交并写入二进制日志时，服务器会自动生成并分配新的 GTID。如果事务回滚或由于其他原因未写入二进制日志，则服务器不会生成和分配 GTID。

+   如果将`gtid_next`设置为有效的 GTID（由 UUID 和事务序列号组成，用冒号分隔），服务器会将该 GTID 分配给您的事务。即使事务未写入二进制日志，或者事务为空，此 GTID 也会被分配并添加到`gtid_executed`。

请注意，在将`gtid_next`设置为特定 GTID 后，事务已提交或回滚后，必须在任何其他语句之前发出显式的`SET @@SESSION.gtid_next`语句。如果不想显式分配更多 GTID，则可以使用此方法将 GTID 值设置回`AUTOMATIC`。

当复制应用程序线程应用复制事务时，它们使用这种技术，将`@@SESSION.gtid_next`显式设置为在原始服务器上分配的复制事务的 GTID。这意味着来自原始服务器的 GTID 被保留，而不是由副本生成和分配新的 GTID。这也意味着即使在副本上禁用了二进制日志记录或副本更新日志，或者事务是无操作或在副本上被过滤掉，GTID 也会被添加到`gtid_executed`。

客户端可以通过将`@@SESSION.gtid_next`设置为特定的 GTID 来模拟一个复制事务，然后执行该事务。这种技术被**mysqlbinlog**用于生成客户端可以重放以保留 GTID 的二进制日志转储。通过客户端提交的模拟复制事务与通过复制应用程序线程提交的复制事务完全等效，在事后无法区分它们。

##### `gtid_purged`系统变量

`gtid_purged`系统变量(`@@GLOBAL.gtid_purged`)中包含了在服务器上已提交但不存在于服务器上任何二进制日志文件中的所有事务的 GTID。`gtid_purged`是`gtid_executed`的子集。`gtid_purged`中包含以下类别的 GTID：

+   在副本上禁用二进制日志记录的复制事务的 GTID。

+   被写入二进制日志文件并已被清除的事务的 GTID。

+   通过语句`SET @@GLOBAL.gtid_purged`显式添加到集合中的 GTID。

您可以更改`gtid_purged`的值，以记录在服务器上已应用某个 GTID 集的事务，尽管这些事务在服务器上的任何二进制日志中都不存在。当您将 GTID 添加到`gtid_purged`时，它们也会被添加到`gtid_executed`中。执行此操作的一个示例用例是，当您在服务器上恢复一个或多个数据库的备份时，但您没有包含服务器上事务的相关二进制日志。在 MySQL 8.0 之前，只有在`gtid_executed`（因此也是`gtid_purged`）为空时才能更改`gtid_purged`的值。从 MySQL 8.0 开始，不再有此限制，您还可以选择是否用指定的 GTID 集替换`gtid_purged`中的整个 GTID 集，或者将指定的 GTID 集添加到已在`gtid_purged`中的 GTID 中。有关如何执行此操作的详细信息，请参阅`gtid_purged`的描述。

当服务器启动时，`gtid_executed` 和 `gtid_purged` 系统变量中的 GTID 集合在初始化时被初始化。每个二进制日志文件都以事件`Previous_gtids_log_event`开头，其中包含了所有先前二进制日志文件中的 GTIDs 集合（由前一个文件的`Previous_gtids_log_event`中的 GTIDs 和前一个文件中每个`Gtid_log_event`的 GTIDs 组成）。最老和最近的二进制日志文件中的`Previous_gtids_log_event`的内容用于计算服务器启动时的`gtid_executed` 和 `gtid_purged` 集合：

+   `gtid_executed` 是由最近的二进制日志文件中的`Previous_gtids_log_event`中的 GTIDs、该二进制日志文件中的事务的 GTIDs 以及`mysql.gtid_executed`表中存储的 GTIDs 的并集计算而得。这个 GTID 集合包含了服务器上已经使用过的 GTIDs（或者明确添加到`gtid_purged`中的 GTIDs），无论它们当前是否在服务器上的二进制日志文件中。它不包括当前在服务器上处理的事务的 GTIDs（`@@GLOBAL.gtid_owned`）。

+   `gtid_purged` 首先通过将最近的二进制日志文件中的`Previous_gtids_log_event`中的 GTIDs 和该二进制日志文件中的事务的 GTIDs 相加来计算。这一步给出了当前或曾经在服务器上的二进制日志中记录的 GTIDs 集合（`gtids_in_binlog`）。接下来，从最老的二进制日志文件中的`Previous_gtids_log_event`中减去`gtids_in_binlog`中的 GTIDs。这一步给出了当前在服务器上的二进制日志中记录的 GTIDs 集合（`gtids_in_binlog_not_purged`）。最后，从`gtid_executed`中减去`gtids_in_binlog_not_purged`。结果是在服务器上已经使用过的 GTIDs 集合，但当前没有在服务器上的二进制日志文件中记录，这个结果用于初始化`gtid_purged`。

如果涉及来自 MySQL 5.7.7 或更早版本的二进制日志在这些计算中，可能会计算出`gtid_executed`和`gtid_purged`的不正确 GTID 集，并且即使稍后重新启动服务器，它们仍然不正确。有关详细信息，请参阅`binlog_gtid_simple_recovery`系统变量的描述，该变量控制如何迭代二进制日志以计算 GTID 集。如果服务器上适用于其中描述的情况之一，请在启动服务器之前在服务器的配置文件中设置`binlog_gtid_simple_recovery=FALSE`。该设置使服务器迭代所有二进制日志文件（而不仅仅是最新和最旧的）以查找 GTID 事件开始出现的位置。如果服务器有大量没有 GTID 事件的二进制日志文件，这个过程可能需要很长时间。

##### 重置 GTID 执行历史

如果需要在服务器上重置 GTID 执行历史，请使用`RESET MASTER`语句。例如，在新的 GTID 启用服务器上执行测试查询以验证复制设置，或者当您想要将新服务器加入到复制组中但它包含一些不被组复制接受的不需要的本地事务时，可能需要执行此操作。

警告

谨慎使用`RESET MASTER`，以避免丢失任何需要的 GTID 执行���史和二进制日志文件。

在执行`RESET MASTER`之前，请确保已备份服务器的二进制日志文件和二进制日志索引文件（如果有），并获取并保存`gtid_executed`系统变量的全局值中保存的 GTID 集（例如，通过执行`SELECT @@GLOBAL.gtid_executed`语句并保存结果）。如果要从该 GTID 集中删除不需要的事务，请使用**mysqlbinlog**检查事务的内容，以确保它们没有价值，不包含必须保存或复制的数据，并且不会导致服务器上的数据更改。

当您执行`RESET MASTER`时，将执行以下重置操作：

+   `gtid_purged`系统变量的值被设置为空字符串（`''`）。

+   全局值（但不是会话值）的`gtid_executed`系统变量被设置为空字符串。

+   `mysql.gtid_executed` 表被清空（参见 mysql.gtid_executed Table）。

+   如果服务器启用了二进制日志记录，则现有的二进制日志文件将被删除，二进制日志索引文件将被清空。

请注意，`RESET MASTER` 是重置 GTID 执行历史记录的方法，即使服务器是一个禁用二进制日志记录的副本。`RESET REPLICA` 对 GTID 执行历史记录没有影响。
