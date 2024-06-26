> 原文：[`dev.mysql.com/doc/refman/8.0/en/thread-pool-tuning.html`](https://dev.mysql.com/doc/refman/8.0/en/thread-pool-tuning.html)

#### 7.6.3.4 线程池调优

本节提供了关于确定线程池性能最佳配置的指导，以事务每秒等指标来衡量。

最重要的是线程池中线程组的数量，可以在服务器启动时使用`--thread-pool-size`选项进行设置；这个值在运行时无法更改。对于这个选项的推荐值取决于主要使用的存储引擎是`InnoDB`还是`MyISAM`：

+   如果主要存储引擎是`InnoDB`，线程池大小的推荐值是主机机器上可用的物理核心数，最大不超过 512。

+   如果主要存储引擎是`MyISAM`，线程池大小应该设置得相对较低。通常情况下，最佳性能在 4 到 8 之间。较高的值可能会对性能产生轻微负面影响，但不会太明显。

线程池插件可以处理的并发事务数量上限由`thread_pool_max_transactions_limit`的值确定。这个系统变量的推荐初始设置是物理核心数乘以 32。您可能需要根据特定工作负载调整这个值；这个值的合理上限是预期的最大并发连接数；`Max_used_connections`状态变量的值可以作为确定这个值的指导。一个好的方法是将`thread_pool_max_transactions_limit`设置为这个值，然后在观察吞吐量的影响时逐渐调整它的值。

线程组中允许的最大查询线程数由`thread_pool_query_threads_per_group`的值确定，可以在运行时进行调整。此值与线程池大小的乘积大致等于可用于处理查询的总线程数。通常获得最佳性能意味着在`thread_pool_query_threads_per_group`和线程池大小之间为您的应用程序找到适当的平衡。较大的`thread_pool_query_threads_per_group`值使得在线程组中同时执行长时间查询并阻塞较短查询的可能性较小，当工作负载包括长时间和短时间运行的查询时。您应该记住，当使用较小的线程池大小值和较大的`thread_pool_query_threads_per_group`值时，每个线程组的连接轮询操作的开销会增加。因此，我们建议将`thread_pool_query_threads_per_group`的起始值设置为`2`；将此变量设置为较低的值通常不会提供任何性能优势。

在正常情况下获得最佳性能，我们还建议将`thread_pool_algorithm`设置为 1 以实现高并发。

此外，`thread_pool_stall_limit`系统变量的值决定了阻塞和长时间运行语句的处理方式。如果所有阻塞 MySQL 服务器的调用都报告给线程池，那么它将始终知道执行线程何时被阻塞，但这并不总是正确的。例如，可能会在未使用线程池回调进行仪器化的代码中发生阻塞。对于这种情况，线程池必须能够识别似乎被阻塞的线程。这是通过由`thread_pool_stall_limit`的值确定的超时来完成的，该超时确保服务器不会完全被阻塞。`thread_pool_stall_limit`的值表示 10 毫秒间隔的数量，因此`600`（最大值）表示 6 秒。

`thread_pool_stall_limit`还可以使线程池处理长时间运行的语句。如果允许长时间运行的语句阻塞线程组，那么分配给该组的所有其他连接都将被阻塞，无法开始执行，直到长时间运行的语句完成。在最坏的情况下，这可能需要几个小时甚至几天。

应选择`thread_pool_stall_limit`的值，使得执行时间超过该值的语句被视为停滞。停滞的语句会产生大量额外开销，因为它们涉及额外的上下文切换，有时甚至涉及额外的线程创建。另一方面，将`thread_pool_stall_limit`参数设置得太高意味着长时间运行的语句会阻塞一些短时间运行的语句，时间超过必要的时间。较短的等待值允许线程更快地启动。较短的值也更有利于避免死锁情况。较长的等待值对包含长时间运行语句的工作负载有用，以避免在当前语句执行时启动太多新语句。

假设一个服务器执行的工作负载中，即使在服务器负载较重时，99.9%的语句在 100ms 内完成，而剩余的语句在 100ms 至 2 小时之间均匀分布。在这种情况下，将`thread_pool_stall_limit`设置为 10（10 × 10ms = 100ms）是有意义的。默认值为 6（60ms），适用于主要执行非常简单语句的服务器。

`thread_pool_stall_limit` 参数可以在运行时更改，以使您能够找到适合服务器工作负载的平衡。假设启用了`tp_thread_group_stats`表，您可以使用以下查询来确定停滞执行语句的比例：

```sql
SELECT SUM(STALLED_QUERIES_EXECUTED) / SUM(QUERIES_EXECUTED)
FROM performance_schema.tp_thread_group_stats;
```

这个数字应尽可能低。为了减少语句停滞的可能性，增加`thread_pool_stall_limit`的值。

当一个语句到达时，它实际开始执行之前可以延迟的最长时间是多少？假设满足以下条件：

+   低优先级队列中有 200 个语句在排队。

+   高优先级队列中有 10 个语句在排队。

+   `thread_pool_prio_kickup_timer` 设置为 10000（10 秒）。

+   `thread_pool_stall_limit` 设置为 100（1 秒）。

在最坏的情况下，这 10 个高优先级语句代表了 10 个长时间执行的事务。因此，在最坏的情况下，没有语句可以移动到高优先级队列，因为它总是包含等待执行的语句。在 10 秒后，新语句有资格被移动到高优先级队列。然而，在它被移动之前，它之前的所有语句也必须被移动。这可能需要另外 2 秒，因为每秒最多可以将 100 个语句移动到高优先级队列。现在当语句到达高优先级队列时，可能会有许多长时间运行的语句在它前面。在最坏的情况下，每一个都会被阻塞，每个语句之间需要 1 秒才能从高优先级队列中检索到下一个语句。因此，在这种情况下，新语句开始执行前需要 222 秒。

这个例子展示了一个应用程序的最坏情况。如何处理取决于应用程序。如果应用程序对响应时间有很高的要求，它很可能应该在更高的级别自行限制用户。否则，它可以使用线程池配置参数来设置某种最大等待时间。
