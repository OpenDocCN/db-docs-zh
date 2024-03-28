# 第十八章 备选存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/storage-engines.html`](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html)

**目录**

18.1 设置存储引擎

18.2 MyISAM 存储引擎

18.2.1 MyISAM 启动选项

18.2.2 键所需的空间

18.2.3 MyISAM 表存储格式

18.2.4 MyISAM 表问题

18.3 MEMORY 存储引擎

18.4 CSV 存储引擎

18.4.1 修复和检查 CSV 表

18.4.2 CSV 限制

18.5 ARCHIVE 存储引擎

18.6 BLACKHOLE 存储引擎

18.7 MERGE 存储引擎

18.7.1 MERGE 表优缺点

18.7.2 MERGE 表问题

18.8 FEDERATED 存储引擎

18.8.1 FEDERATED 存储引擎概述

18.8.2 如何创建 FEDERATED 表

18.8.3 FEDERATED 存储引擎注意事项和提示

18.8.4 FEDERATED 存储引擎资源

18.9 EXAMPLE 存储引擎

18.10 其他存储引擎

18.11 MySQL 存储引擎架构概述

18.11.1 可插拔存储引擎架构

18.11.2 通用数据库服务器层

存储引擎是 MySQL 组件，用于处理不同表类型的 SQL 操作。`InnoDB` 是默认和最通用的存储引擎，Oracle 建议除了专用用例外，都使用它来创建表。（在 MySQL 8.0 中，`CREATE TABLE` 语句默认创建 `InnoDB` 表。）

MySQL 服务器使用可插拔存储引擎架构，允许存储引擎在运行中加载和卸载。

要确定服务器支持哪些存储引擎，请使用 `SHOW ENGINES` 语句。`Support` 列中的值指示引擎是否可用。`YES`、`NO` 或 `DEFAULT` 的值表示引擎可用、不可用或可用且当前设置为默认存储引擎。

```sql
mysql> SHOW ENGINES\G
*************************** 1\. row ***************************
      Engine: PERFORMANCE_SCHEMA
     Support: YES
     Comment: Performance Schema
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 2\. row ***************************
      Engine: InnoDB
     Support: DEFAULT
     Comment: Supports transactions, row-level locking, and foreign keys
Transactions: YES
          XA: YES
  Savepoints: YES
*************************** 3\. row ***************************
      Engine: MRG_MYISAM
     Support: YES
     Comment: Collection of identical MyISAM tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 4\. row ***************************
      Engine: BLACKHOLE
     Support: YES
     Comment: /dev/null storage engine (anything you write to it disappears)
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 5\. row ***************************
      Engine: MyISAM
     Support: YES
     Comment: MyISAM storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
...
```

本章涵盖了特定用途的 MySQL 存储引擎的使用案例。它不涵盖默认的 `InnoDB` 存储引擎或 `NDB` 存储引擎，这些在 Chapter 17, *The InnoDB Storage Engine* 和 Chapter 25, *MySQL NDB Cluster 8.0* 中有介绍。对于高级用户，它还包含了可插拔存储引擎架构的描述（请参见 Section 18.11, “Overview of MySQL Storage Engine Architecture”）。

有关商业 MySQL Server 二进制版本提供的功能信息，请访问 MySQL 网站上的[*MySQL Editions*](https://www.mysql.com/products/)。可用的存储引擎可能取决于您使用的 MySQL 版本。

关于 MySQL 存储引擎常见问题的答案，请参见 Section A.2, “MySQL 8.0 FAQ: Storage Engines”。

## MySQL 8.0 支持的存储引擎

+   `InnoDB`：MySQL 8.0 中的默认存储引擎。`InnoDB` 是 MySQL 的事务安全（ACID 兼容）存储引擎，具有提交、回滚和崩溃恢复功能，以保护用户数据。`InnoDB` 行级锁定（不升级为更粗粒度的锁）和 Oracle 风格的一致性非锁定读取增加了多用户并发性和性能。`InnoDB` 将用户数据存储在聚集索引中，以减少基于主键的常见查询的 I/O。为了保持数据完整性，`InnoDB` 还支持 `FOREIGN KEY` 外键完整性约束。有关 `InnoDB` 的更多信息，请参见 Chapter 17, *The InnoDB Storage Engine*。

+   `MyISAM`：这些表占用空间小。表级锁定 限制了读/写工作负载的性能，因此通常用于 Web 和数据仓库配置中的只读或读多写少的工作负载。

+   `Memory`：将所有数据存储在 RAM 中，用于快速访问需要快速查找非关键数据的环境。这个引擎以前被称为 `HEAP` 引擎。它的使用案例正在减少；`InnoDB` 通过其缓冲池内存区提供了一种通用且耐用的方式来将大部分或全部数据保存在内存中，而 `NDBCLUSTER` 为庞大的分布式数据集提供了快速的键值查找。

+   `CSV`：它的表实际上是带有逗号分隔值的文本文件。CSV 表允许你以 CSV 格式导入或导出数据，与读写相同格式的脚本和应用程序交换数据。因为 CSV 表没有索引，通常在正常操作期间将数据保存在`InnoDB`表中，仅在导入或导出阶段使用 CSV 表。

+   `归档`：这些紧凑的、无索引的表用于存储和检索大量很少被引用的历史、归档或安全审计信息。

+   `黑洞`：黑洞存储引擎接受但不存储数据，类似于 Unix 的`/dev/null`设备。查询总是返回一个空集。这些表可以在复制配置中使用，其中 DML 语句被发送到副本服务器，但源服务器不保留自己的数据副本。

+   `NDB`（也称为`NDBCLUSTER`）：这个集群数据库引擎特别适用于需要最高可用性和可用性的应用程序。

+   `合并`：使 MySQL DBA 或开发人员可以逻辑地将一系列相同的`MyISAM`表分组，并将它们引用为一个对象。适用于数据仓库等 VLDB 环境。

+   `联合`：提供了将不同的 MySQL 服务器链接起来，从许多物理服务器创建一个逻辑数据库的能力。非常适合分布式或数据仓库环境。

+   `示例`：这个引擎在 MySQL 源代码中作为一个示例，展示了如何开始编写新的存储引擎。主要是为开发人员感兴趣。这个存储引擎是一个“存根”，什么也不做。你可以用这个引擎创建表，但不能在其中存储或检索数据。

你不必限制整个服务器或模式使用相同的存储引擎。你可以为任何表指定存储引擎。例如，一个应用程序可能主要使用`InnoDB`表，其中一个`CSV`表用于将数据导出到电子表格，以及一些`MEMORY`表用于临时工作空间。

**选择存储引擎**

MySQL 提供的各种存储引擎是针对不同用例设计的。以下表格提供了一些 MySQL 提供的存储引擎的概述，表格后面跟着澄清说明。

**表 18.1 存储引擎特性摘要**

| 特性 | MyISAM | Memory | InnoDB | Archive | NDB |
| --- | --- | --- | --- | --- | --- |
| B-tree 索引 | 是 | 是 | 是 | 否 | 否 |
| 备份/时间点恢复（注 1） | 是 | 是 | 是 | 是 | 是 |
| 集群数据库支持 | 否 | 否 | 否 | 否 | 是 |
| 聚集索引 | 否 | 否 | 是 | 否 | 否 |
| 压缩数据 | 是（注 2） | 否 | 是 | 是 | 否 |
| 数据缓存 | 否 | 不适用 | 是 | 否 | 是 |
| 加密数据 | 是（注 3） | 是（注 3） | 是（注 4） | 是（注 3） | 是（注 5） |
| 外键支持 | 否 | 否 | 是 | 否 | 是 |
| 全文搜索索引 | 是 | 否 | 是（注 6） | 否 | 否 |
| 地理空间数据类型支持 | 是 | 否 | 是 | 是 | 是 |
| 地理空间索引支持 | 是 | 否 | 是（注 7） | 否 | 否 |
| 哈希索引 | 否 | 是 | 否（注 8） | 否 | 是 |
| 索引缓存 | 是 | 不适用 | 是 | 否 | 是 |
| 锁定粒度 | 表 | 表 | 行 | 行 | 行 |
| MVCC | 否 | 否 | 是 | 否 | 否 |
| 复制支持（注 1） | 是 | 有限（注 9） | 是 | 是 | 是 |
| 存储限制 | 256TB | RAM | 64TB | 无 | 384EB |
| T 树索引 | 否 | 否 | 否 | 否 | 是 |
| 事务 | 否 | 否 | 是 | 否 | 是 |
| 更新数据字典的统计信息 | 是 | 是 | 是 | 是 | 是 |
| 特性 | MyISAM | Memory | InnoDB | Archive | NDB |

**注：**

1\. 在服务器端实现，而不是在存储引擎中。

2\. 仅当使用压缩行格式时，支持压缩的 MyISAM 表。使用压缩行格式的 MyISAM 表是只读的。

3\. 通过加密函数在服务器端实现。

4\. 通过加密函数在服务器端实现；在 MySQL 5.7 及更高版本中，支持数据静态加密。

5\. 通过加密函数在服务器端实现；NDB 8.0.22 起支持加密的 NDB 备份；NDB 8.0.29 及更高版本支持透明的 NDB 文件系统加密。

6\. MySQL 5.6 及更高版本支持全文索引。

7\. MySQL 5.7 及更高版本支持地理空间索引。

8\. InnoDB 在内部利用哈希索引来实现其自适应哈希索引功能。

9\. 请参见本节后面的讨论。
