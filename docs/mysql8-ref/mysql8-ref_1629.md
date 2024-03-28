> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-transactions.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-transactions.html)

#### 25.2.7.3 NDB 集群中与事务处理相关的限制

在处理事务方面，NDB 集群存在一些限制。这些包括以下内容：

+   **事务隔离级别。** `NDBCLUSTER` 存储引擎仅支持 `READ COMMITTED` 事务隔离级别。（例如，`InnoDB` 支持 `READ COMMITTED`、`READ UNCOMMITTED`、`REPEATABLE READ` 和 `SERIALIZABLE`。）您应该记住，`NDB` 在每行基础上实现了 `READ COMMITTED`；当读取请求到达存储该行的数据节点时，返回的是该时刻该行的最后提交版本。

    未提交数据永远不会返回，但是当修改多行的事务与读取相同行的事务同时提交时，执行读取的事务可能会观察到这些行中的不同行的“之前”值、“之后”值或两者，这是因为给定行读取请求可以在另一个事务提交之前或之后处理。

    为确保给定事务仅读取之前或之后的值，您可以使用 `SELECT ... LOCK IN SHARE MODE` 强制施加行锁。在这种情况下，锁将保持直到拥有事务提交。使用行锁还可能导致以下问题：

    +   锁等待超时错误频率增加，并发性降低

    +   由于读取需要提交阶段，事务处理开销增加

    +   可能耗尽可用并发锁数量的可能性，这由 `MaxNoOfConcurrentOperations` 限制

    `NDB` 在所有读取操作中使用 `READ COMMITTED`，除非使用诸如 `LOCK IN SHARE MODE` 或 `FOR UPDATE` 等修饰符。`LOCK IN SHARE MODE` 导致使用共享行锁；`FOR UPDATE` 导致使用独占行锁。唯一键读取会被 `NDB` 自动升级锁以确保自洽读取；`BLOB` 读取也会为了一致性而使用额外的锁定。

    参见 第 25.6.8.4 节，“NDB 集群备份故障排除”，了解 NDB 集群的事务隔离级别实现如何影响 `NDB` 数据库的备份和恢复。

+   **事务和 BLOB 或 TEXT 列。** `NDBCLUSTER`仅在 MySQL 可见的表中存储使用 MySQL 的任何`BLOB`或`TEXT`数据类型的列值的一部分；`BLOB`或`TEXT`的其余部分存储在一个不可访问的单独内部表中。这会引发两个相关问题，您在执行包含这些类型列的表上的`SELECT`语句时应该注意：

    1.  对于从 NDB Cluster 表中的任何`SELECT`：如果`SELECT`包括`BLOB`或`TEXT`列，则`READ COMMITTED`事务隔离级别会转换为带读锁的读取。这样做是为了保证一致性。

    1.  对于任何使用唯一键查找来检索使用任何`BLOB`或`TEXT`数据类型的列的`SELECT`，并且在事务内执行的情况下，表上会持有一个共享读锁，直到事务要么提交要么中止。

        对于使用索引或表扫描的查询，即使针对具有`BLOB`或`TEXT`列的`NDB`表，也不会出现此问题。

        例如，考虑以下`CREATE TABLE`语句定义的表`t`：

        ```sql
        CREATE TABLE t (
            a INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
            b INT NOT NULL,
            c INT NOT NULL,
            d TEXT,
            INDEX i(b),
            UNIQUE KEY u(c)
        ) ENGINE = NDB,
        ```

        对`t`的以下查询会导致共享读锁，因为它使用了唯一键查找：

        ```sql
        SELECT * FROM t WHERE c = 1;
        ```

        然而，这里展示的四个查询中没有一个会导致共享读锁：

        ```sql
        SELECT * FROM t WHERE b = 1;

        SELECT * FROM t WHERE d = '1';

        SELECT * FROM t;

        SELECT b,c WHERE a = 1;
        ```

        这是因为在这四个查询中，第一个使用索引扫描，第二和第三使用表扫描，而第四个虽然使用主键查找，但不检索任何`BLOB`或`TEXT`列的值。

        通过避免检索`BLOB`或`TEXT`列的唯一键查找查询，或者在无法避免这类查询的情况下，尽快提交事务，可以帮助最小化共享读锁的问题。

+   **唯一键查找和事务隔离。** 使用隐藏索引表在`NDB`中实现唯一索引，该表在内部维护。当使用唯一索引访问用户创建的`NDB`表时，首先读取隐藏索引表以找到然后用于读取用户创建的表的主键。为了避免在这种双重读取操作期间修改索引，对在隐藏索引表中找到的行进行锁定。当更新用户创建的`NDB`表中唯一索引引用的行时，由执行更新的事务对隐藏索引表施加排他锁。这意味着对同一（用户创建的）`NDB`表的任何读取操作都必须等待更新完成。即使读取操作的事务级别为`READ COMMITTED`，也是如此。

    可以用于绕过潜在阻塞读取的一种解决方法是强制 SQL 节点在执行读取时忽略唯一索引。这可以通过在读取表时使用`IGNORE INDEX`索引提示作为`SELECT`语句的一部分来实现（参见 Section 10.9.4, “Index Hints”）。因为 MySQL 服务器为在`NDB`中创建的每个唯一索引创建了一个阴影有序索引，这样可以读取有序索引，避免唯一索引访问锁定。结果读取与按主键提交的读取一样一致，在读取行时返回最后提交的值。

    通过有序索引进行读取会较少有效地利用集群资源，并可能具有较高的延迟。

    也可以通过查询范围而不是唯一值来避免使用唯一索引进行访问。

+   **回滚。** 没有部分事务，也没有部分事务回滚。重复键或类似错误会导致整个事务回滚。

    这种行为与其他事务存储引擎（如`InnoDB`）不同，后者可能会回滚单个语句。

+   **事务和内存使用。** 如本章其他地方所述，NDB Cluster 不擅长处理大型事务；最好执行一些包含少量操作的小事务，而不是尝试包含大量操作的单个大型事务。除其他考虑外，大型事务需要非常大量的内存。因此，一些 MySQL 语句的事务行为受到影响，如下列表所述：

    +   在`NDB`表上使用`TRUNCATE TABLE`时不具有事务性。如果`TRUNCATE TABLE`未能清空表格，则必须重复运行直到成功。

    +   `DELETE FROM`（即使没有`WHERE`子句）*是*事务性的。对于包含大量行的表，您可能会发现通过使用多个`DELETE FROM ... LIMIT ...`语句来“分块”删除操作可以提高性能。如果您的目标是清空表格，则可能希望改用`TRUNCATE TABLE`。

    +   **LOAD DATA 语句。** `LOAD DATA` 在`NDB`表上使用时不具有事务性。

        重要提示

        在执行`LOAD DATA`语句时，`NDB`引擎以不规则的间隔执行提交，以便更好地利用通信网络。无法提前知道这些提交何时发生。

    +   **ALTER TABLE 和事务。** 在作为`ALTER TABLE`的一部分复制`NDB`表时，复制的创建是非事务性的。（无论如何，当复制被删除时，此操作会被回滚。）

+   **事务和 COUNT() 函数。** 在使用 NDB Cluster Replication 时，无法保证副本上`COUNT()`函数的事务一致性。换句话说，在源上执行一系列语句（`INSERT`、`DELETE`或两者）以在单个事务中更改表中的行数时，在副本上执行`SELECT COUNT(*) FROM *`table`*`查询可能会产生中间结果。这是因为`SELECT COUNT(...)`可能执行脏读，并不是`NDB`存储引擎中的错误。（有关更多信息，请参见 Bug #31321。）
