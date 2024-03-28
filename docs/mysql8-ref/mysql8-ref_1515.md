> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-configuring-consistency-guarantees.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-configuring-consistency-guarantees.html)

#### 20.5.3.2 配置事务一致性保证

尽管 事务同步点 部分解释了概念上有两个可以选择的同步点：读取或写入时，但这些术语是一种简化，Group Replication 中使用的术语是：*事务执行前* 和 *事务执行后*。一致性级别对由组处理的只读（RO）和读写（RW）事务可能产生不同的影响，正如本节所示。

+   如何选择一致性级别

+   一致性级别的影响

+   一致性对主选举的影响

+   一致性规则下允许的查询

以下列表显示了在 Group Replication 中可以使用 `group_replication_consistency` 变量配置的可能一致性级别，按照事务一致性保证递增的顺序排列：

+   `EVENTUAL`

    无论是 RO 还是 RW 事务在执行时都不会等待前置事务被应用。这是在添加 `group_replication_consistency` 变量之前 Group Replication 的行为。一个 RW 事务不会等待其他成员应用事务。这意味着一个事务在其他成员之前可能在一个成员上被外部化。这也意味着在主故障转移的情况下，新的主节点可以在之前的主节点的所有事务都被应用之前接受新的 RO 和 RW 事务。RO 事务可能导致过时的值，RW 事务可能由于冲突而导致回滚。

+   `BEFORE_ON_PRIMARY_FAILOVER`

    具有新选举的主要成员并正在应用旧主要成员的积压的新 RO 或 RW 事务被保留（未应用），直到任何积压被应用。这确保了主要故障转移发生时，无论是故意还是不经意，客户端始终看到主要成员上的最新值。这保证了一致性，但意味着客户端必须能够处理正在应用积压时的延迟。通常，此延迟应该很小，但这取决于积压的大小。

+   `BEFORE`

    RW 事务等待所有先前事务完成后再应用。RO 事务等待所有先前事务完成后再执行。这确保了此事务通过仅影响事务的延迟来读取最新值。通过仅在 RO 事务上使用同步，可以减少每个 RW 事务上的同步开销。此一致性级别还包括`BEFORE_ON_PRIMARY_FAILOVER`提供的一致性保证。

+   `AFTER`

    RW 事务等待直到其更改已应用于所有其他成员。此值对 RO 事务没有影响。此模式确保当本地成员上提交事务时，任何后续事务都会读取已写入的值或任何组成员上更近的值。在主要用于 RO 操作的组中使用此模式，以确保一旦提交，已应用的 RW 事务就会被应用到所有地方。您的应用程序可以使用此模式来确保后续读取获取包括最新写入的最新数据。通过仅在 RW 事务上使用同步，可以减少每个 RO 事务上的同步开销。此一致性级别还包括`BEFORE_ON_PRIMARY_FAILOVER`提供的一致性保证。

+   `BEFORE_AND_AFTER`

    RW 事务等待 1) 所有先前事务完成后再应用和 2) 直到其更改已应用于其他成员。RO 事务等待所有先前事务完成后再执行。此一致性级别还包括`BEFORE_ON_PRIMARY_FAILOVER`提供的一致性保证。

`BEFORE` 和 `BEFORE_AND_AFTER` 一致性级别都可以用于 RO 和 RW 事务。`AFTER` 一致性级别对 RO 事务没有影响，因为它们不生成更改。

##### 如何选择一致性级别

不同的一致性级别为 DBA 和开发人员提供了灵活性，他们可以使用这些级别来设置基础设施；开发人员可以根据其应用程序的要求选择最适合的一致性级别。以下场景展示了如何根据您使用组的方式选择一致性保证级别：

+   *场景 1* 您希望负载均衡读取而不必担心过时读取，您的组写入操作明显少于组读取操作。在这种情况下，您应选择`AFTER`。

+   *场景 2* 你有一个应用了大量写入的数据集，并且希望偶尔进行读取而不必担心读取过时数据。在这种情况下，你应该选择`BEFORE`。

+   *场景 3* 你希望工作负载中的特定事务始终从组中读取最新数据，因此每当敏感数据更新（例如文件的凭据或类似数据）时，你希望强制读取始终读取最新值。在这种情况下，你应该选择`BEFORE`。

+   *场景 4* 你有一个主要包含只读（RO）数据的组，你希望你的读写（RW）事务一旦提交就在所有地方应用，以便后续读取是在包括最新写入的最新数据上进行的，而且你不需要在每个 RO 事务上进行同步，而只需在 RW 事务上进行。在这种情况下，你应该选择`AFTER`。

+   *场景 5* 你有一个主要包含只读数据的组，你希望你的读写（RW）事务始终从组中读取最新数据，并且一旦提交就在所有地方应用，以便后续读取是在包括最新写入的最新数据上进行的，而且你不需要在每个只读（RO）事务上进行同步，而只需在 RW 事务上进行。在这种情况下，你应该选择`BEFORE_AND_AFTER`。

你有自由选择强制执行一致性级别的范围。这很重要，因为如果在全局范围设置一致性级别，可能会对组性能产生负面影响。因此，你可以通过在不同范围使用`group_replication_consistency`系统变量来配置组的一致性级别。

要在当前会话上强制执行一致性级别，请使用会话范围：

```sql
> SET @@SESSION.group_replication_consistency= 'BEFORE';
```

要在所有会话上强制执行一致性级别，请使用全局范围：

```sql
> SET @@GLOBAL.group_replication_consistency= 'BEFORE';
```

在特定会话上设置一致性级别的可能性使你能够利用以下场景：

+   *场景 6* 给定系统处理几个不需要强一致性级别的指令，但一种指令确实需要强一致性：管理对文档的访问权限。在这种情况下，系统更改访问权限并希望确保所有客户端看到正确的权限。你只需要在这些指令上`SET @@SESSION.group_replication_consistency= ‘AFTER’`，并将其他指令保留在全局范围设置为`EVENTUAL`。

+   *场景 7* 在与场景 6 中描述的相同系统上，每天都需要执行一些分析处理的指令，并且因此需要始终读取最新数据。为了实现这一点，你只需要在该特定指令上`SET @@SESSION.group_replication_consistency= ‘BEFORE’`。

总结一下，你不需要以特定的一致性级别运行所有事务，特别是如果只有一些事务实际上需要它。

请注意，在组复制中，所有读写事务都是完全有序的，因此，即使您将一致性级别设置为`AFTER`，对于当前会话，此事务也会等待直到其更改在所有成员上应用，这意味着等待此事务和所有可能在次要队列中的前置事务。实际上，一致性级别`AFTER`会等待一切，直到包括此事务在内。

##### 一致性级别的影响

另一种分类一致性级别的方式是根据对组的影响来进行，即，一致性级别对其他成员的影响。

除了在事务流中排序外，一致性级别`BEFORE`仅影响本地成员。也就是说，它不需要与其他成员协调，也不会对其事务产生影响。换句话说，`BEFORE`仅影响使用它的事务。

一致性级别`AFTER`和`BEFORE_AND_AFTER`对在其他成员上执行的并发事务确实会产生副作用。如果具有`EVENTUAL`一致性级别的事务在执行具有`AFTER`或`BEFORE_AND_AFTER`的事务时开始，其他成员的事务将等待，直到`AFTER`事务在该成员上提交，即使其他成员的事务具有`EVENTUAL`一致性级别。换句话说，`AFTER`和`BEFORE_AND_AFTER`影响*所有*`ONLINE`组成员。

为了进一步说明这一点，想象一个有 3 个成员 M1、M2 和 M3 的组。在成员 M1 上，一个客户端发出：

```sql
> SET @@SESSION.group_replication_consistency= AFTER;
> BEGIN;
> INSERT INTO t1 VALUES (1);
> COMMIT;
```

然后，在应用上述事务时，成员 M2 上的一个客户端发出：

```sql
> SET SESSION group_replication_consistency= EVENTUAL;
```

在这种情况下，即使第二个事务的一致性级别是`EVENTUAL`，因为它在第一个事务已经在 M2 上处于提交阶段时开始执行，第二个事务必须等待第一个事务完成提交，然后才能执行。

你只能在`ONLINE`成员上使用一致性级别`BEFORE`、`AFTER`和`BEFORE_AND_AFTER`，尝试在其他状态的成员上使用它们会导致会话错误。

一致性级别不是`EVENTUAL`的事务会等待直到达到由`wait_timeout`值配置的超时，其默认值为 8 小时。如果超时，将抛出`ER_GR_HOLD_WAIT_TIMEOUT`错误。

##### 一致性对初选的影响

本节描述了一个群组的一致性级别如何影响已选出新主节点的单主群组。这样的群组会自动检测故障并调整活跃成员的视图，换句话说，成员配置。此外，如果一个群组以单主模式部署，每当群组成员发生变化时，都会执行检查以检测群组中是否仍有主成员。如果没有，则从次要成员列表中选择一个新的主成员。通常，这被称为次要晋升。

鉴于系统能够自动检测故障并自动重新配置自身，用户可能也期望一旦晋升发生，新的主节点在数据方面与旧主节点完全相同。换句话说，用户可能期望在他能够从新主节点读取和写入时，新主节点上没有待应用的复制事务。实际上，用户可能期望一旦他的应用故障切换到新主节点，甚至暂时也不会有机会读取旧数据或写入旧数据记录。

当在群组上激活并正确调整流量控制时，在晋升后立即从新选出的主节点暂时读取陈旧数据的机会很小，因为不应该有积压，或者如果有积压，那么应该很小。此外，您可能会有一个代理或中间件层，在晋升后管理应用程序对主节点的访问并在该级别强制执行一致性标准。如果您的群组成员使用的是 MySQL 8.0.14 或更高版本，您可以使用 `group_replication_consistency` 变量指定新主节点在晋升后的行为，该变量控制新选出的主节点是否阻止读取和写入，直到积压完全应用或者如果它的行为类似于运行 MySQL 8.0.13 或更早版本的成员。如果在有待应用的积压的新选出的主节点上设置了 `group_replication_consistency` 选项为 `BEFORE_ON_PRIMARY_FAILOVER`，并且在新主节点仍在应用积压时发出事务，则传入事务将被阻止，直到积压完全应用。因此，以下异常情况被防止：

+   对于只读和读写事务，不会出现陈旧的读取。这可以防止陈旧的读取通过新主节点外部化到应用程序中。

+   由于与仍在等待应用的积压中的复制读写事务发生写-写冲突，因此读写事务不会出现虚假回滚。

+   在读写事务中没有读取偏差，例如：

    ```sql
    > BEGIN;
    > SELECT x FROM t1; -- x=1 because x=2 is in the backlog;
    > INSERT x INTO t2;
    > COMMIT;
    ```

    这个查询不应该引起冲突，但会写入过时的数值。

总结一下，当 `group_replication_consistency` 设置为 `BEFORE_ON_PRIMARY_FAILOVER` 时，您选择优先考虑一致性而不是可用性，因为每当选举新的主要成员时，读写操作都会被保留。这是在配置组时必须考虑的权衡。还应记住，如果流量控制正常工作，backlog 应该是最小的。请注意，更高的一致性级别 `BEFORE`、`AFTER` 和 `BEFORE_AND_AFTER` 也包括 `BEFORE_ON_PRIMARY_FAILOVER` 提供的一致性保证。

为了确保组提供相同的一致性级别，无论哪个成员被提升为主要成员，组的所有成员都应该将 `BEFORE_ON_PRIMARY_FAILOVER`（或更高的一致性级别）持久化到其配置中。例如，在每个成员上执行：

```sql
> SET PERSIST group_replication_consistency='BEFORE_ON_PRIMARY_FAILOVER';
```

这确保了所有成员都以相同的方式行事，并且配置在成员重新启动后仍然保留。

事务不能永远保持挂起状态，如果持续时间超过 `wait_timeout`，则会返回一个 ER_GR_HOLD_WAIT_TIMEOUT 错误。

##### 在一致性规则下允许的查询

在使用 `BEFORE_ON_PRIMARY_FAILOVER` 一致性级别时，虽然所有写操作都被保留，但并非所有读操作都被阻塞，以确保您仍然可以在升级后的服务器应用 backlog 时进行检查。这对于调试、监控、可观察性和故障排除非常有用。一些不修改数据的查询是允许的，例如以下内容：

+   `SHOW` 语句 - 从 MySQL 8.0.27 开始，此处仅限于不依赖于数据，仅依赖于状态和配置的语句，如下所列

+   `SET` 语句

+   不使用表或可加载函数的 `DO` 语句

+   `EMPTY` 语句

+   `USE` 语句

+   使用针对 `performance_schema` 和 `sys` 数据库的 `SELECT` 语句

+   使用针对 `infoschema` 数据库中的 `PROCESSLIST` 表的 `SELECT` 语句

+   不使用表或可加载函数的 `SELECT` 语句

+   `STOP GROUP_REPLICATION` 语句

+   `SHUTDOWN` 语句

+   `RESET PERSIST` 语句

允许在 MySQL 8.0.27 中使用的`SHOW`语句包括`SHOW VARIABLES, SHOW PROCESSLIST, SHOW STATUS, SHOW ENGINE INNODB LOGS, SHOW ENGINE INNODB STATUS, SHOW ENGINE INNODB MUTEX, SHOW MASTER STATUS, SHOW REPLICA STATUS, SHOW CHARACTER SET, SHOW COLLATION, SHOW BINARY LOGS, SHOW OPEN TABLES, SHOW REPLICAS, SHOW BINLOG EVENTS, SHOW WARNINGS, SHOW ERRORS, SHOW ENGINES, SHOW PRIVILEGES, SHOW PROCEDURE STATUS, SHOW FUNCTION STATUS, SHOW PLUGINS, SHOW EVENTS, SHOW PROFILE, SHOW PROFILES,` 和 `SHOW RELAYLOG EVENTS`。
