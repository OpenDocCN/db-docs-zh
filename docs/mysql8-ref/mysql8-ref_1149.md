# 17.2 InnoDB 和 ACID 模型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-acid.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-acid.html)

ACID 模型是一组强调对商业数据和关键应用程序重要性的可靠性方面的数据库设计原则。MySQL 包括诸如 `InnoDB` 存储引擎等组件，严格遵循 ACID 模型，以确保数据不会被损坏，并且结果不会受到异常条件（如软件崩溃和硬件故障）的扭曲。当您依赖符合 ACID 标准的特性时，您无需重新发明一致性检查和崩溃恢复机制。在您拥有额外的软件保护、超可靠的硬件或者可以容忍少量数据丢失或不一致的应用程序的情况下，您可以调整 MySQL 设置，以交换一些 ACID 可靠性以获得更大的性能或吞吐量。

以下各节讨论了 MySQL 特性，特别是 `InnoDB` 存储引擎，如何与 ACID 模型的各个类别交互：

+   **A**: 原子性。

+   **C**: 一致性。

+   **I:**: 隔离性。

+   **D**: 持久性。

### 原子性

ACID 模型中的**原子性**主要涉及 `InnoDB` 事务。相关的 MySQL 特性包括：

+   `autocommit` 设置。

+   `COMMIT` 语句。

+   `ROLLBACK` 语句。

### 一致性

ACID 模型中的**一致性**主要涉及内部 `InnoDB` 处理以保护数据免受崩溃的影响。相关的 MySQL 特性包括：

+   `InnoDB` 双写缓冲区。参见 Section 17.6.4, “Doublewrite Buffer”.

+   `InnoDB` 崩溃恢复。参见 InnoDB Crash Recovery.

### 隔离性

ACID 模型中的**隔离性**主要涉及 `InnoDB` 事务，特别是适用于每个事务的隔离级别。相关的 MySQL 特性包括：

+   `autocommit` 设置。

+   事务隔离级别和 `SET TRANSACTION` 语句。参见 Section 17.7.2.1, “Transaction Isolation Levels”.

+   `InnoDB` 锁定 的低级细节。细节可以在 `INFORMATION_SCHEMA` 表中查看（参见 Section 17.15.2, “InnoDB INFORMATION_SCHEMA Transaction and Locking Information”）以及 Performance Schema 的 `data_locks` 和 `data_lock_waits` 表。

### 持久性

ACID 模型中的**持久性**方面涉及 MySQL 软件功能与您特定硬件配置的交互。由于取决于 CPU、网络和存储设备的功能，这方面是最复杂的，无法提供具体的指导方针。（这些指导方针可能采取“购买新硬件”的形式。）相关的 MySQL 功能包括：

+   `InnoDB` 双写缓冲区。参见 Section 17.6.4, “Doublewrite Buffer”。

+   `innodb_flush_log_at_trx_commit` 变量。

+   `sync_binlog` 变量。

+   `innodb_file_per_table` 变量。

+   存储设备中的写入缓冲区，如磁盘驱动器、固态硬盘或 RAID 阵列。

+   存储设备中的带电池备份缓存。

+   用于运行 MySQL 的操作系统，特别是对 `fsync()` 系统调用的支持。

+   为所有运行 MySQL 服务器和存储 MySQL 数据的计算机服务器和存储设备提供电力保护的不间断电源（UPS）。

+   你的备份策略，如备份频率和类型，以及备份保留期限。

+   对于分布式或托管数据应用程序，MySQL 服务器硬件所在的数据中心的特定特性，以及数据中心之间的网络连接。
