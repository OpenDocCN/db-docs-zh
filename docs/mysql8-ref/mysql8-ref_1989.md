# 第二十九章 MySQL 性能模式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema.html)

**目录**

29.1 Performance Schema 快速入门

29.2 Performance Schema 构建配置

29.3 Performance Schema 启动配置

29.4 Performance Schema 运行时配置

29.4.1 Performance Schema 事件定时

29.4.2 Performance Schema 事件过滤

29.4.3 事件预过滤

29.4.4 按仪器进行预过滤

29.4.5 按对象进行预过滤

29.4.6 按线程进行预过滤

29.4.7 按消费者进行预过滤

29.4.8 示例消费者配置

29.4.9 命名仪器或消费者以进行过滤操作

29.4.10 确定什么被仪器化

29.5 Performance Schema 查询

29.6 Performance Schema 仪器命名约定

29.7 Performance Schema 状态监控

29.8 Performance Schema 原子和分子事件

29.9 Performance Schema 当前和历史事件表

29.10 Performance Schema 语句摘要和抽样

29.11 Performance Schema 通用表特性

29.12 Performance Schema 表描述

29.12.1 Performance Schema 表参考

29.12.2 Performance Schema 设置表

29.12.3 Performance Schema 实例表

29.12.4 Performance Schema 等待事件表

29.12.5 Performance Schema 阶段事件表

29.12.6 Performance Schema 语句事件表

29.12.7 Performance Schema 事务表

29.12.8 Performance Schema 连接表

29.12.9 Performance Schema 连接属性表

29.12.10 Performance Schema User-Defined Variable Tables

29.12.11 Performance Schema Replication Tables

29.12.12 性能模式 NDB 集群表

29.12.13 Performance Schema Lock Tables

29.12.14 性能模式系统变量表

29.12.15 性能模式状态变量表

29.12.16 性能模式线程池表

29.12.17 性能模式防火墙表

29.12.18 Performance Schema Keyring Tables

29.12.19 性能模式克隆表

29.12.20 性能模式摘要表

29.12.21 Performance Schema Miscellaneous Tables

29.13 性能模式选项和变量参考

29.14 性能模式命令选项

29.15 性能模式系统变量

29.16 Performance Schema Status Variables

29.17 The Performance Schema Memory-Allocation Model

29.18 性能模式和插件

29.19 使用性能模式诊断问题

29.19.1 使用性能模式进行查询分析

29.19.2 获取父事件信息

29.20 性能模式的限制

MySQL 性能模式是用于监视 MySQL 服务器在低级别执行的功能。性能模式具有以下特点：

+   性能模式提供了一种在运行时检查服务器内部执行的方式。它是使用`PERFORMANCE_SCHEMA`存储引擎和`performance_schema`数据库实现的。性能模式主要关注性能数据。这与`INFORMATION_SCHEMA`不同，后者用于元数据的检查。

+   Performance Schema 监视服务器事件。一个“事件”是服务器执行需要时间的任何操作，并且已经被仪器化以便收集时间信息。一般来说，事件可以是函数调用，等待操作系统，SQL 语句执行的阶段（如解析或排序），或整个语句或一组语句。事件收集提供了关于同步调用（如互斥锁）、文件和表 I/O、表锁等服务器和几个存储引擎的信息。

+   Performance Schema 事件与写入服务器二进制日志的事件（描述数据修改）和事件调度程序事件（存储程序的一种类型）是不同的。

+   Performance Schema 事件特定于给定的 MySQL 服务器实例。Performance Schema 表被视为服务器本地的，对它们的更改不会被复制或写入二进制日志。

+   当前事件可用，以及事件历史和摘要。这使您能够确定仪器化活动被执行的次数以及所花费的时间。事件信息可用于显示特定线程的活动，或与特定对象（如互斥锁或文件）相关联的活动。

+   `PERFORMANCE_SCHEMA`存储引擎使用服务器源代码中的“仪器化点”收集事件数据。

+   收集的事件存储在`performance_schema`数据库的表中。这些表可以像其他表一样使用`SELECT`语句进行查询。

+   Performance Schema 配置可以通过通过 SQL 语句更新`performance_schema`数据库中的表来动态修改。配置更改会立即影响数据收集。

+   Performance Schema 中的表是使用内存而不使用持久性磁盘存储的表。内容在服务器启动时重新填充，并在服务器关闭时丢弃。

+   监控在 MySQL 支持的所有平台上都可用。

    可能存在一些限制：计时器类型可能因平台而异。适用于存储引擎的仪器可能并非所有存储引擎都实现。每个第三方引擎的仪器化是引擎维护者的责任。另请参阅 Section 29.20, “Restrictions on Performance Schema”。

+   数据收集通过修改服务器源代码以添加仪器实现。与其他功能（如复制或事件调度程序）不同，Performance Schema 没有与之关联的单独线程。

Performance Schema 旨在在对服务器性能影响最小的情况下提供有关服务器执行的有用信息。实现遵循以下设计目标：

+   激活性能模式不会改变服务器行为。例如，它不会导致线程调度发生变化，也不会导致查询执行计划（如`EXPLAIN`所示）发生变化。

+   服务器监控持续进行，几乎没有额外开销。激活性能模式不会使服务器无法使用。

+   解析器保持不变。没有新的关键字或语句。

+   即使性能模式在内部失败，服务器代码的执行仍然正常进行。

+   当在事件收集初始阶段或稍后在事件检索期间执行处理时有选择时，优先考虑使收集更快。这是因为收集是持续进行的，而检索是按需进行的，甚至可能根本不会发生。

+   大多数性能模式表都有索引，这使得优化器可以访问除全表扫描之外的执行计划。有关更多信息，请参见第 10.2.4 节，“优化性能模式查询”。

+   添加新的仪表点很容易。

+   仪表化是有版本的。如果仪表化实现发生变化，先前仪表化的代码仍然可以正常工作。这有利于第三方插件的开发人员，因为不需要升级每个插件以与最新性能模式更改保持同步。

注意

MySQL `sys`模式是一组对象，提供方便访问性能模式收集的数据。`sys`模式默认安装。有关使用说明，请参见第三十章，“MySQL sys 模式”。
