> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-unsupported.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-unsupported.html)

#### 25.2.7.6 NDB Cluster 中不支持或缺失的功能

其他存储引擎支持的许多功能对`NDB`表不受支持。在 NDB Cluster 中尝试使用这些功能之一不会导致错误本身; 但是，应用程序可能会出现错误，因为它们期望这些功能得到支持或强制执行。引用这些功能的语句，即使被`NDB`有效忽略，也必须在语法和其他方面有效。

+   **索引前缀。** 对`NDB`表不支持索引前缀。如果在语句中（如`CREATE TABLE`、`ALTER TABLE`或`CREATE INDEX`）的索引规范中使用前缀，则`NDB`不会创建前缀。

    包含索引前缀的语句，并创建或修改`NDB`表，仍必须在语法上有效。例如，以下语句始终失败，显示错误 1089 不正确的前缀键; 使用的键部分不是字符串，使用的长度比键部分长，或存储引擎不支持唯一前缀键，无论存储引擎如何：

    ```sql
    CREATE TABLE t1 (
        c1 INT NOT NULL,
        *c2 VARCHAR(100),
        INDEX i1 (c2(500))* );
    ```

    这是由于 SQL 语法规则导致没有索引可以具有比自身更大的前缀。

+   **保存点和回滚。** 保存点和回滚到保存点被忽略，就像在`MyISAM`中一样。

+   **提交的持久性。** 磁盘上没有持久的提交。提交是复制的，但不能保证在提交时日志被刷新到磁盘上。

+   **复制。** 不支持基于语句的复制。在设置集群复制时，请使用`--binlog-format=ROW`（或`--binlog-format=MIXED`）。有关更多信息，请参见第 25.7 节“NDB Cluster 复制”。

    使用全局事务标识符（GTID）进行复制与 NDB Cluster 不兼容，并且在 NDB Cluster 8.0 中不受支持。在使用`NDB`存储引擎时不要启用 GTID，因为这很可能会导致问题，甚至导致 NDB Cluster 复制失败。

    NDB Cluster 不支持半同步复制。

+   **生成列。** `NDB`存储引擎不支持虚拟生成列上的索引。

    与其他存储引擎一样，您可以在存储的生成列上创建索引，但您应该记住，`NDB` 使用`DataMemory`来存储生成列以及`IndexMemory`来存储索引。有关示例，请参见 NDB Cluster 中的 JSON 列和间接索引。

    NDB Cluster 将存储的生成列的更改写入二进制日志，但不记录对虚拟列的更改。这不应影响 NDB Cluster 复制或`NDB`与其他 MySQL 存储引擎之间的复制。

注意

有关在`NDB`中处理事务限制的更多信息，请参见第 25.2.7.3 节“NDB Cluster 中与事务处理相关的限制”。
