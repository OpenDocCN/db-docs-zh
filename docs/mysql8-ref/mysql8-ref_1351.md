> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-gtids-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-gtids-functions.html)

#### 19.1.3.8 操作 GTID 的存储函数示例

本节提供了使用 MySQL 提供的一些内置函数创建的存储函数示例（请参阅第二十七章，*存储对象*），这些函数可用于与基于 GTID 的复制一起使用，列在此处：

+   `GTID_SUBSET()`：显示一个 GTID 集合是否是另一个的子集。

+   `GTID_SUBTRACT()`：返回一个 GTID 集合中不在另一个中的 GTID。

+   `WAIT_FOR_EXECUTED_GTID_SET()`: 等待给定 GTID 集合中的所有事务被执行。

有关刚列出的函数的更多信息，请参阅第 14.18.2 节，“与全局事务标识符（GTID）一起使用的函数”。

请注意，在这些存储函数中，分隔符命令已被用于将 MySQL 语句分隔符更改为竖线，如下所示：

```sql
mysql> delimiter |
```

本节中显示的所有存储函数都将 GTID 集合的字符串表示作为参数，因此在与它们一起使用时，GTID 集合必须始终带引号。

此函数在两个 GTID 集合相同时返回非零（true），即使它们的格式不同：

```sql
CREATE FUNCTION GTID_IS_EQUAL(gs1 LONGTEXT, gs2 LONGTEXT)
  RETURNS INT
  RETURN GTID_SUBSET(gs1, gs2) AND GTID_SUBSET(gs2, gs1)
|
```

此函数在两个 GTID 集合不相交时返回非零（true）：

```sql
CREATE FUNCTION GTID_IS_DISJOINT(gs1 LONGTEXT, gs2 LONGTEXT)
RETURNS INT
  RETURN GTID_SUBSET(gs1, GTID_SUBTRACT(gs1, gs2))
|
```

此函数在两个 GTID 集合不相交且`sum`为它们的并集时返回非零（true）：

```sql
CREATE FUNCTION GTID_IS_DISJOINT_UNION(gs1 LONGTEXT, gs2 LONGTEXT, sum LONGTEXT)
RETURNS INT
  RETURN GTID_IS_EQUAL(GTID_SUBTRACT(sum, gs1), gs2) AND
         GTID_IS_EQUAL(GTID_SUBTRACT(sum, gs2), gs1)
|
```

此函数返回 GTID 集合的规范形式，全部大写，无空格和无重复，UUID 按字母顺序排列，间隔按数字顺序排列：

```sql
CREATE FUNCTION GTID_NORMALIZE(gs LONGTEXT)
RETURNS LONGTEXT
  RETURN GTID_SUBTRACT(gs, '')
|
```

此函数返回两个 GTID 集合的并集：

```sql
CREATE FUNCTION GTID_UNION(gs1 LONGTEXT, gs2 LONGTEXT)
RETURNS LONGTEXT
  RETURN GTID_NORMALIZE(CONCAT(gs1, ',', gs2))
|
```

此函数返回两个 GTID 集合的交集。

```sql
CREATE FUNCTION GTID_INTERSECTION(gs1 LONGTEXT, gs2 LONGTEXT)
RETURNS LONGTEXT
  RETURN GTID_SUBTRACT(gs1, GTID_SUBTRACT(gs1, gs2))
|
```

此函数返回两个 GTID 集合之间的对称差异，即存在于`gs1`中但不存在于`gs2`中的 GTID，以及存在于`gs2`中但不存在于`gs1`中的 GTID。

```sql
CREATE FUNCTION GTID_SYMMETRIC_DIFFERENCE(gs1 LONGTEXT, gs2 LONGTEXT)
RETURNS LONGTEXT
  RETURN GTID_SUBTRACT(CONCAT(gs1, ',', gs2), GTID_INTERSECTION(gs1, gs2))
|
```

此函数从 GTID 集合中删除所有具有指定来源的 GTID，并返回剩余的 GTID（如果有的话）。 UUID 是服务器的标识符，事务通常是在`server_uuid`中的值。

```sql
CREATE FUNCTION GTID_SUBTRACT_UUID(gs LONGTEXT, uuid TEXT)
RETURNS LONGTEXT
  RETURN GTID_SUBTRACT(gs, CONCAT(UUID, ':1-', (1 << 63) - 2))
|
```

此函数充当前一个函数的反向；它仅返回来自具有指定标识符（UUID）的服务器的 GTID 集合中的 GTID。

```sql
CREATE FUNCTION GTID_INTERSECTION_WITH_UUID(gs LONGTEXT, uuid TEXT)
RETURNS LONGTEXT
  RETURN GTID_SUBTRACT(gs, GTID_SUBTRACT_UUID(gs, uuid))
|
```

**示例 19.1 验证副本是否最新**

内置函数`GTID_SUBSET()`和`GTID_SUBTRACT()`可用于检查副本是否至少应用了源应用的每个事务。

要使用`GTID_SUBSET()`执行此检查，请在副本上执行以下语句：

```sql
SELECT GTID_SUBSET(*source_gtid_executed*, *replica_gtid_executed*);
```

如果返回值为 `0`（false），则意味着 *`source_gtid_executed`* 中的一些 GTID 不在 *`replica_gtid_executed`* 中，并且副本尚未应用在源上应用的事务，这意味着副本不是最新的。

要使用 `GTID_SUBTRACT()` 执行相同的检查，在副本上执行以下语句：

```sql
SELECT GTID_SUBTRACT(*source_gtid_executed*, *replica_gtid_executed*);
```

此语句返回在 *`source_gtid_executed`* 中但不在 *`replica_gtid_executed`* 中的任何 GTID。如果返回任何 GTID，则表示源已应用了一些副本尚未应用的事务，因此副本不是最新的。

**示例 19.2 备份和恢复场景**

存储函数 `GTID_IS_EQUAL()`、`GTID_IS_DISJOINT()` 和 `GTID_IS_DISJOINT_UNION()` 可用于验证涉及多个数据库和服务器的备份和恢复操作。在此示例场景中，`server1` 包含数据库 `db1`，`server2` 包含数据库 `db2`。目标是将数据库 `db2` 复制到 `server1`，并且在 `server1` 上的结果应该是两个数据库的并集。使用的步骤是使用 **mysqldump** 备份 `server2`，然后在 `server1` 上恢复此备份。

运行 **mysqldump** 时，如果使用了 `--set-gtid-purged` 参数设置为 `ON` 或 `AUTO`（默认值），输出将包含一个 `SET @@GLOBAL.gtid_purged` 语句，该语句将从 `server2` 的 `gtid_executed` 集合添加到 `server1` 的 `gtid_purged` 集合中。`gtid_purged` 包含在给定服务器上已提交但在服务器上任何二进制日志文件中不存在的所有事务的 GTID。当将数据库 `db2` 复制到 `server1` 时，必须将在 `server2` 上提交的事务的 GTID（这些事务不在 `server1` 的二进制日志文件中）添加到 `server1` 的 `gtid_purged` 中，以使集合完整。

存储函数可用于协助此场景中的以下步骤：

+   使用 `GTID_IS_EQUAL()` 验证备份操作是否为 `SET @@GLOBAL.gtid_purged` 语句计算了正确的 GTID 集合。在 `server2` 上，从 **mysqldump** 输出中提取该语句，并将 GTID 集合存储到一个本地变量中，例如 `$gtid_purged_set`。然后执行以下语句：

    ```sql
    server2> SELECT GTID_IS_EQUAL($gtid_purged_set, @@GLOBAL.gtid_executed);
    ```

    如果结果为 1，则两个 GTID 集合相等，且集合已正确计算。

+   使用`GTID_IS_DISJOINT()`验证**mysqldump**输出中的 GTID 集与`server1`上的`gtid_executed`集不重叠。在两个服务器上存在相同的 GTID 会导致将数据库`db2`复制到`server1`时出现错误。要检查，在`server1`上，从输出中提取并存储`gtid_purged`到一个本地变量中，然后执行以下语句：

    ```sql
    server1> SELECT GTID_IS_DISJOINT($gtid_purged_set, @@GLOBAL.gtid_executed);
    ```

    如果结果为 1，则两个 GTID 集之间没有重叠，因此不存在重复的 GTID。

+   使用`GTID_IS_DISJOINT_UNION()`验证还原操作是否在`server1`上导致正确的 GTID 状态。在还原备份之前，在`server1`上执行以下语句获取现有的`gtid_executed`集：

    ```sql
    server1> SELECT @@GLOBAL.gtid_executed;
    ```

    将结果存储在本地变量`$original_gtid_executed`中，以及如前所述，将`gtid_purged`中的集合存储在另一个本地变量中。当从`server2`恢复备份到`server1`时，执行以下语句验证 GTID 状态：

    ```sql
    server1> SELECT 
     ->   GTID_IS_DISJOINT_UNION($original_gtid_executed,
     ->                          $gtid_purged_set,
     ->                          @@GLOBAL.gtid_executed);
    ```

    如果结果为`1`，则存储的函数已验证来自`server1`的原始`gtid_executed`集（`$original_gtid_executed`）和从`server2`添加的`gtid_purged`集（`$gtid_purged_set`）没有重叠，并且`server1`上更新的`gtid_executed`集现在包括来自`server1`的先前`gtid_executed`集以及来自`server2`的`gtid_purged`集，这是期望的结果。确保在`server1`上发生任何进一步的事务之前进行此检查，否则`gtid_executed`中的新事务将导致其失败。

**示例 19.3 选择最新的副本进行手动故障转移**

存储函数`GTID_UNION()`可用于从一组副本中识别最新的副本，以便在源服务器意外停止后执行手动故障转移操作。如果一些副本正在经历复制延迟，此存储函数可用于计算最新的副本，而无需等待所有副本应用其现有的中继日志，从而最小化故障转移时间。该函数可以返回每个副本上的`gtid_executed`与副本接收到的事务集的并集，该事务集记录在性能模式`replication_connection_status`表中。您可以比较这些结果，找出哪个副本的事务记录是最新的，即使并非所有事务都已提交。

在每个副本上，通过执行以下语句计算完整的事务记录：

```sql
SELECT GTID_UNION(RECEIVED_TRANSACTION_SET, @@GLOBAL.gtid_executed)
    FROM performance_schema.replication_connection_status
    WHERE channel_name = 'name';
```

然后可以比较每个副本的结果，看哪个副本具有最新的事务记录，并将此副本用作新的源。

**示例 19.4 检查副本上的多余交易**

存储函数`GTID_SUBTRACT_UUID()`可用于检查副本是否接收了未来自其指定来源或来源的交易。如果有，则可能存在复制设置问题，或代理、路由器或负载均衡器存在问题。此函数通过从指定起始服务器的 GTID 集合中移除所有 GTID，并返回剩余的 GTID（如果有）来工作。

对于具有单个来源的副本，请发出以下语句，提供起始来源的标识符，通常与`server_uuid`相同：

```sql
SELECT GTID_SUBTRACT_UUID(@@GLOBAL.gtid_executed, server_uuid_of_source);
```

如果结果不为空，则返回的交易是未来自指定来源的额外交易。

对于多源拓扑中的副本，请在函数调用中包含每个来源的服务器 UUID，如下所示：

```sql
SELECT 
  GTID_SUBTRACT_UUID(GTID_SUBTRACT_UUID(@@GLOBAL.gtid_executed,
                                        server_uuid_of_source_1),
                                        server_uuid_of_source_2);
```

如果结果不为空，则返回的交易是未来自任何指定来源的额外交易。

**示例 19.5 验证复制拓扑中的服务器是否为只读**

存储函数`GTID_INTERSECTION_WITH_UUID()`可用于验证服务器是否未发起任何 GTID 并处于只读状态。该函数仅返回来自具有指定标识符的服务器的 GTID 集合中的那些 GTID。如果在此服务器的`gtid_executed`中列出的任何交易使用服务器自己的标识符，则服务器本身发起了这些交易。您可以在服务器上发出以下语句进行检查：

```sql
SELECT GTID_INTERSECTION_WITH_UUID(@@GLOBAL.gtid_executed, my_server_uuid);
```

**示例 19.6 在多源复制中验证额外的副本**

存储函数`GTID_INTERSECTION_WITH_UUID()`可用于查找附加到多源复制设置的副本是否已应用来自一个特定源的所有事务。在这种情况下，`source1`和`source2`都是源和副本，并相互复制。`source2`还有自己的副本。如果`source2`配置为`log_replica_updates=ON`，则副本还会接收并应用来自`source1`的事务，但如果`source2`使用`log_replica_updates=OFF`，则不会这样做。无论哪种情况，我们目前只想知道副本是否与`source2`保持同步。在这种情况下，`GTID_INTERSECTION_WITH_UUID()`可用于识别`source2`发起的事务，丢弃`source2`从`source1`复制的事务。然后可以使用内置函数`GTID_SUBSET()`将结果与副本上的`gtid_executed`集进行比较。如果副本与`source2`保持同步，则副本上的`gtid_executed`集包含交集集中的所有事务（源自`source2`的事务）。

要执行此检查，请将`source2`的`gtid_executed`值和服务器 UUID 以及副本中的`gtid_executed`值存储到用户变量中，如下所示：

```sql
source2> SELECT @@GLOBAL.gtid_executed INTO @source2_gtid_executed;

source2> SELECT @@GLOBAL.server_uuid INTO @source2_server_uuid;

replica> SELECT @@GLOBAL.gtid_executed INTO @replica_gtid_executed;
```

然后使用`GTID_INTERSECTION_WITH_UUID()`和`GTID_SUBSET()`，并将这些变量作为输入，如下所示：

```sql
SELECT 
  GTID_SUBSET(
    GTID_INTERSECTION_WITH_UUID(@source2_gtid_executed,
                                @source2_server_uuid),
                                @replica_gtid_executed);
```

来自`source2`的服务器标识符（`@source2_server_uuid`）与`GTID_INTERSECTION_WITH_UUID()`一起使用，仅识别并返回那些在`source2`上发起的 GTID，省略那些在`source1`上发起的。然后，使用`GTID_SUBSET()`将生成的 GTID 集与副本上的所有已执行 GTID 集进行比较。如果此语句返回非零（true），则从`source2`识别的所有 GTID（第一个输入集）也在副本的`gtid_executed`中找到，这意味着副本已接收并执行了所有源自`source2`的事务。
