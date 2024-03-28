# 29.19 使用性能模式诊断问题

> 译文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-examples.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-examples.html)

29.19.1 使用性能模式进行查询分析

29.19.2 获取父事件信息

性能模式是帮助数据库管理员通过进行实际测量而不是“猜测”来进行性能调优的工具。本节演示了使用性能模式进行此目的的一些方法。这里的讨论依赖于事件过滤的使用，该过滤在 Section 29.4.2, “性能模式事件过滤”中有描述。

以下示例提供了一种方法，您可以使用该方法分析可重复的问题，例如调查性能瓶颈。首先，您应该有一个性能被认为“太慢”并需要优化的可重复用例，并且应该启用所有仪器（完全不进行预过滤）。

1.  运行用例。

1.  使用性能模式表，分析性能问题的根本原因。这种分析在很大程度上依赖于后过滤。

1.  对于已排除的问题领域，禁用相应的仪器。例如，如果分析显示问题与特定存储引擎中的文件 I/O 无关，则禁用该引擎的文件 I/O 仪器。然后截断历史和摘要表以删除先前收集的事件。

1.  重复步骤 1 中的过程。

    随着每次迭代，性能模式输出，特别是`events_waits_history_long`表，包含的由于不重要的仪器引起的“噪音”越来越少，而且由于该表具有固定大小，包含的与手头问题分析相关的数据越来越多。

    随着每次迭代，调查应该越来越接近问题的根本原因，因为“信号/噪音”比例改善，使分析变得更容易。

1.  一旦确定了性能瓶颈的根本原因，采取适当的纠正措施，例如：

    +   调整服务器参数（缓存大小、内存等）。

    +   通过不同方式编写查询来调整查询，

    +   调整数据库架构（表、索引等）。

    +   调整代码（仅适用于存储引擎或服务器开发人员）。

1.  重新从步骤 1 开始，以查看对性能的更改产生的影响。

`mutex_instances.LOCKED_BY_THREAD_ID` 和 `rwlock_instances.WRITE_LOCKED_BY_THREAD_ID` 列对于调查性能瓶颈或死锁非常重要。这是通过性能模式仪器实现的：

1.  假设线程 1 因等待互斥锁而被阻塞。

1.  您可以确定线程正在等待什么：

    ```sql
    SELECT * FROM performance_schema.events_waits_current
    WHERE THREAD_ID = *thread_1*;
    ```

    查询结果显示线程正在等待互斥锁 A，位于`events_waits_current.OBJECT_INSTANCE_BEGIN`。

1.  你可以确定哪个线程正在持有互斥锁 A：

    ```sql
    SELECT * FROM performance_schema.mutex_instances
    WHERE OBJECT_INSTANCE_BEGIN = *mutex_A*;
    ```

    查询结果显示线程 2 正在持有互斥锁 A，如`mutex_instances.LOCKED_BY_THREAD_ID`中所示。

1.  你可以看到线程 2 在做什么：

    ```sql
    SELECT * FROM performance_schema.events_waits_current
    WHERE THREAD_ID = *thread_2*;
    ```
