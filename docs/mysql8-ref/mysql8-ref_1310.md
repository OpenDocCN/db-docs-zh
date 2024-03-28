# 18.3 MEMORY 存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/memory-storage-engine.html`](https://dev.mysql.com/doc/refman/8.0/en/memory-storage-engine.html)

`MEMORY` 存储引擎（以前称为 `HEAP`）创建内容存储在内存中的特殊用途表。由于数据容易受到崩溃、硬件问题或停电的影响，只能将这些表用作临时工作区或从其他表中提取数据的只读缓存。

**表 18.4 MEMORY 存储引擎特性**

| 特性 | 支持 |
| --- | --- |
| **B 树索引** | 是 |
| **备份/时间点恢复**（在服务器中实现，而不是在存储引擎中。） | 是 |
| **集群数据库支持** | 否 |
| **聚集索引** | 否 |
| **压缩数据** | 否 |
| **数据缓存** | 不适用 |
| **加密数据** | 是（通过加密函数在服务器中实现。） |
| **外键支持** | 否 |
| **全文搜索索引** | 否 |
| **地理空间数据类型支持** | 否 |
| **地理空间索引支持** | 否 |
| **哈希索引** | 是 |
| **索引缓存** | 不适用 |
| **锁定粒度** | 表 |
| **MVCC** | 否 |
| **复制支持**（在服务器中实现，而不是在存储引擎中。） | 有限（请参见本节后面的讨论。） |
| **存储限制** | RAM |
| **T 树索引** | 否 |
| **事务** | 否 |
| **更新数据字典统计信息** | 是 |
| 特性 | 支持 |

+   何时使用 MEMORY 或 NDB 集群

+   分区

+   性能特性

+   MEMORY 表的特点

+   MEMORY 表的 DDL 操作

+   索引

+   用户创建和临时表

+   加载数据

+   MEMORY 表和复制

+   管理内存使用

+   其他资源

### 何时使用 MEMORY 或 NDB 集群

寻求部署使用 `MEMORY` 存储引擎处理重要、高可用或频繁更新数据的应用程序的开发人员应考虑 NDB Cluster 是否更合适。`MEMORY` 引擎的典型用例包括以下特点：

+   涉及瞬态、非关键数据的操作，如会话管理或缓存。当 MySQL 服务器停止或重新启动时，`MEMORY` 表中的数据将丢失。

+   内存存储以实现快速访问和低延迟。数据量完全可以放入内存，而不会导致操作系统交换虚拟内存页。

+   读取为主或只读数据访问模式（有限更新）。

NDB Cluster 提供与 `MEMORY` 引擎相同的功能，性能水平更高，并提供 `MEMORY` 不支持的其他功能：

+   行级锁定和多线程操作可减少客户端之间的低争用。

+   即使包含写操作的语句混合，也具有可伸缩性。

+   可选的磁盘支持操作以实现数据持久性。

+   无共享架构和无单点故障的多主机操作，实现 99.999%的可用性。

+   跨节点自动数据分布；应用程序开发人员无需设计自定义分片或分区解决方案。

+   支持变长数据类型（包括`BLOB`和`TEXT`），`MEMORY` 不支持。

### 分区

`MEMORY` 表不支持分区。

### 性能特征

`MEMORY` 性能受限于单线程执行和处理更新时的表锁定开销所导致的争用。当负载增加时，特别是包含写操作的语句混合时，这限制了可伸缩性。

尽管 `MEMORY` 表采用内存处理，但在繁忙服务器上，对于通用查询或读写工作负载，它们不一定比`InnoDB`表更快。特别是，执行更新涉及的表锁定可能会减慢多个会话中对 `MEMORY` 表的并发使用。

根据在 `MEMORY` 表上执行的查询类型，您可以创建索引作为默认哈希数据结构（用于基于唯一键查找单个值）或通用的 B 树数据结构（用于涉及等式、不等式或范围运算符（如小于或大于）的所有查询）。以下部分说明了创建这两种索引的语法。一个常见的性能问题是在 B 树索引更有效的工作负载中使用默认哈希索引。

### `MEMORY` 表的特点

`MEMORY` 存储引擎不在磁盘上创建任何文件。表定义存储在 MySQL 数据字典中。

`MEMORY` 表具有以下特点：

+   `MEMORY` 表的空间是以小块分配的。表使用 100% 的动态哈希插入。不需要溢出区域或额外的键空间。不需要额外的空间用于空闲列表。删除的行被放入一个链表中，在你向表中插入新数据时被重用。`MEMORY` 表也没有通常与哈希表中的删除加插入相关的问题。

+   `MEMORY` 表使用固定长度的行存储格式。可变长度类型如 `VARCHAR` 使用固定长度存储。

+   `MEMORY` 表不能包含 `BLOB` 或 `TEXT` 列。

+   `MEMORY` 包括对 `AUTO_INCREMENT` 列的支持。

+   非临时的 `MEMORY` 表在所有客户端之间共享，就像任何其他非临时表一样。

### `MEMORY` 表的 DDL 操作

要创建一个 `MEMORY` 表，在 `CREATE TABLE` 语句中指定 `ENGINE=MEMORY` 子句。

```sql
CREATE TABLE t (i INT) ENGINE = MEMORY;
```

如引擎名称所示，`MEMORY` 表存储在内存中。它们默认使用哈希索引，使得单值查找非常快速，并且非常适用于创建临时表。然而，当服务器关闭时，所有存储在 `MEMORY` 表中的行都会丢失。表本身会继续存在，因为它们的定义存储在 MySQL 数据字典中，但在服务器重新启动时它们是空的。

这个例子展示了如何创建、使用和移除一个 `MEMORY` 表：

```sql
mysql> CREATE TABLE test ENGINE=MEMORY
           SELECT ip,SUM(downloads) AS down
           FROM log_table GROUP BY ip;
mysql> SELECT COUNT(ip),AVG(down) FROM test;
mysql> DROP TABLE test;
```

`MEMORY` 表的最大大小受 `max_heap_table_size` 系统变量限制，默认值为 16MB。要为 `MEMORY` 表强制不同的大小限制，改变这个变量的值。对于 `CREATE TABLE`，或随后的 `ALTER TABLE` 或 `TRUNCATE TABLE`，生效的值将用于表的生命周期。服务器重新启动也会将现有 `MEMORY` 表的最大大小设置为全局 `max_heap_table_size` 值。你可��像本节后面描述的那样为单个表设置大小。

### 索引

`MEMORY` 存储引擎支持 `HASH` 和 `BTREE` 索引。你可以通过添加 `USING` 子句来指定给定索引使用的其中一个，如下所示：

```sql
CREATE TABLE lookup
    (id INT, INDEX USING HASH (id))
    ENGINE = MEMORY;
CREATE TABLE lookup
    (id INT, INDEX USING BTREE (id))
    ENGINE = MEMORY;
```

关于 B 树和哈希索引的一般特性，请参阅 第 10.3.1 节，“MySQL 如何使用索引”。

`MEMORY` 表每个表最多可以有 64 个索引，每个索引最多有 16 列，最大键长度为 3072 字节。

如果`MEMORY`表的哈希索引具有高度的键重复（许多包含相同值的索引条目），那么影响键值的表更新和所有删除操作将显着变慢。这种减速程度与重复程度成正比（或者与索引基数成反比）。您可以使用`BTREE`索引来避免这个问题。

`MEMORY`表可以具有非唯一键。（这是哈希索引实现的一个不常见的特性。）

索引的列可以包含`NULL`值。

### 用户创建的临时表

`MEMORY`表内容存储在内存中，这是`MEMORY`表与服务器在处理查询时动态创建的内部临时表共享的属性。然而，这两种类型的表在以下方面有所不同：`MEMORY`表不受存储转换的影响，而内部临时表受到影响：

+   如果内部临时表变得太大，服务器会自动将其转换为磁盘存储，如第 10.4.4 节，“MySQL 中的内部临时表使用”所述。

+   用户创建的`MEMORY`表永远不会转换为磁盘表。

### 加载数据

在 MySQL 服务器启动时填充`MEMORY`表，您可以使用`init_file`系统变量。例如，您可以将诸如`INSERT INTO ... SELECT`或`LOAD DATA`之类的语句放入文件中，从持久数据源加载表，并使用`init_file`命名文件。请参见第 7.1.8 节，“服务器系统变量”和第 15.2.9 节，“LOAD DATA 语句”。

### MEMORY 表和复制

当复制源服务器关闭并重新启动时，其`MEMORY`表变为空。为了将此效果复制到副本，源在启动后第一次使用给定的`MEMORY`表时，它会记录一个事件，通知副本必须通过向二进制日志写入该表的`DELETE`或（从 MySQL 8.0.22 开始）`TRUNCATE TABLE`语句来清空该表。当副本服务器关闭并重新启动时，其`MEMORY`表也变为空，并且它会向自己的二进制日志写入一个`DELETE`或（从 MySQL 8.0.22 开始）`TRUNCATE TABLE`语句，这些语句会传递给任何下游副本。

当您在复制拓扑中使用`MEMORY`表时，在某些情况下，源表和副本表可能会有所不同。有关处理这些情况以防止过时读取或错误的信息，请参阅 Section 19.5.1.21，“复制和 MEMORY 表”。

### 管理内存使用

服务器需要足够的内存来同时维护所有正在使用的`MEMORY`表。

如果您从`MEMORY`表中删除单个行，则不会回收内存。只有在删除整个表时才会回收内存。先前用于已删除行的内存将在同一表中为新行重新使用。当您不再需要其内容时，要释放`MEMORY`表使用的所有内存，请执行`DELETE`或`TRUNCATE TABLE`以删除所有行，或使用`DROP TABLE`完全删除表。要释放已删除行使用的内存，请使用`ALTER TABLE ENGINE=MEMORY`强制表重建。

在`MEMORY`表中，一个行所需的内存通过以下表达式计算：

```sql
SUM_OVER_ALL_BTREE_KEYS(*max_length_of_key* + sizeof(char*) * 4)
+ SUM_OVER_ALL_HASH_KEYS(sizeof(char*) * 2)
+ ALIGN(*length_of_row*+1, sizeof(char*))
```

`ALIGN()`表示一个向上取整的因子，使行长度成为`char`指针大小的精确倍数。在 32 位机器上，`sizeof(char*)`为 4，在 64 位机器上为 8。

如前所述，`max_heap_table_size`系统变量设置了`MEMORY`表的最大大小限制。要控制各个表的最大大小，需在创建每个表之前设置此变量的会话值。（除非您打算将该值用于所有客户端创建的`MEMORY`表，否则不要更改全局`max_heap_table_size`值。）以下示例创建了两个`MEMORY`表，分别具有 1MB 和 2MB 的最大大小：

```sql
mysql> SET max_heap_table_size = 1024*1024;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE t1 (id INT, UNIQUE(id)) ENGINE = MEMORY;
Query OK, 0 rows affected (0.01 sec)

mysql> SET max_heap_table_size = 1024*1024*2;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE t2 (id INT, UNIQUE(id)) ENGINE = MEMORY;
Query OK, 0 rows affected (0.00 sec)
```

如果服务器重新启动，两个表都会恢复到服务器的全局`max_heap_table_size`值。

您还可以在`CREATE TABLE`语句中为`MEMORY`表指定`MAX_ROWS`表选项，以提供关于您计划在其中存储的行数的提示。这不会使表增长超过`max_heap_table_size`值，该值仍然作为最大表大小的约束。为了能够灵活使用`MAX_ROWS`，请将`max_heap_table_size`设置至少与您希望每个`MEMORY`表能够增长的值一样高。

### 附加资源

一个专门致力于`MEMORY`存储引擎的论坛可在[`forums.mysql.com/list.php?92`](https://forums.mysql.com/list.php?92)找到。
