# 25.5.3 **ndbmtd** — 多线程的 NDB 集群数据节点守护程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbmtd.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbmtd.html)

**ndbmtd**是**ndbd**的多线程版本，用于处理使用`NDBCLUSTER`存储引擎的所有表中的数据的进程。**ndbmtd**旨在用于具有多个 CPU 核心的主机计算机。除非另有说明，否则**ndbmtd**的功能方式与**ndbd**相同；因此，在本节中，我们重点关注**ndbmtd**与**ndbd**的区别，并且您应该参考第 25.5.1 节，“ndbd — NDB 集群数据节点守护程序”，获取有关运行适用于数据节点进程的单线程和多线程版本的 NDB 集群数据节点的附加信息。

与**ndbd**一起使用的命令行选项和配置参数也适用于**ndbmtd**。有关这些选项和参数的更多信息，请参见第 25.5.1 节，“ndbd — NDB 集群数据节点守护程序”和第 25.4.3.6 节，“定义 NDB 集群数据节点”。

**ndbmtd**") 也与**ndbd**兼容文件系统。换句话说，运行**ndbd**的数据节点可以停止，用**ndbmtd**")替换二进制文件，然后重新启动而不会丢失任何数据。（但是，在执行此操作时，如果希望**ndbmtd**")以多线程方式运行，则必须在重新启动节点之前确保`MaxNoOfExecutionThreads`设置为适当的值。）同样，可以通过停止节点然后在多线程二进制文件位置启动**ndbd**来简单地用**ndbmtd**")替换**ndbmtd**")二进制文件。在切换两者之间时，不需要使用`--initial`启动数据节点二进制文件。

使用**ndbmtd**")与使用**ndbd**有两个关键区别：

1.  因为**ndbmtd**")默认以单线程模式运行（即，它的行为类似于**ndbd**），所以必须配置它以使用多个线程。可以通过在`config.ini`文件中为`MaxNoOfExecutionThreads`配置参数或`ThreadConfig`配置参数设置适当的值来实现。使用`MaxNoOfExecutionThreads`更简单，但`ThreadConfig`提供更多灵活性。有关这些配置参数及其用法的更多信息，请参阅多线程配置参数（ndbmtd）。

1.  由于**ndbmtd**")进程中的关键错误，跟**ndbd**失败生成的跟踪文件有些许不同。这些差异将在接下来的几段中详细讨论。

与**ndbd**类似，**ndbmtd**")生成一组日志文件，这些文件放置在`config.ini`配置文件中指定的目录中，即`DataDir`。除了跟踪文件外，这些文件以与**ndbd**生成方式相同的方式生成，并具有与**ndbd**生成的文件相同的名称。

在发生关键错误时，**ndbmtd**")会生成描述错误发生前发生的情况的跟踪文件。这些文件可以在数据节点的`DataDir`中找到，对于 NDB Cluster 开发和支持团队来说，这些文件对问题分析非常有用。每个**ndbmtd**")线程都会生成一个跟踪文件。这些文件的命名模式如下：

```sql
ndb_*node_id*_trace.log.*trace_id*_t*thread_id*,
```

在这个模式中，*`node_id`*代表集群中数据节点的唯一节点 ID，*`trace_id`*是一个跟踪序列号，*`thread_id`*是线程 ID。例如，如果作为 NDB Cluster 数据节点运行的**ndbmtd**")进程失败，节点 ID 为 3，并且`MaxNoOfExecutionThreads`等于 4，那么在数据节点的数据目录中会生成四个跟踪文件。如果这是该节点第一次失败，那么这些文件的名称将是`ndb_3_trace.log.1_t1`，`ndb_3_trace.log.1_t2`，`ndb_3_trace.log.1_t3`和`ndb_3_trace.log.1_t4`。在内部，这些跟踪文件遵循与**ndbd**跟踪文件相同的格式。

**ndbd**退出代码和消息是在数据节点进程意外关闭时生成的，也被**ndbmtd**")使用。请参阅数据节点错误消息，以查看这些消息的列表。

注意

可以在同一 NDB 集群中的不同数据节点上同时使用**ndbd**和**ndbmtd**")。然而，这样的配置并未经过广泛测试；因此，我们目前不建议在生产环境中这样做。
