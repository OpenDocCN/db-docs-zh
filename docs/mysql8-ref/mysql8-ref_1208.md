# 17.8.4 为 InnoDB 配置线程并发性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-performance-thread_concurrency.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-thread_concurrency.html)

`InnoDB`使用操作系统线程来处理来自用户事务的请求。（事务在提交或回滚之前可能向`InnoDB`发出许多请求。）在具有多核处理器的现代操作系统和服务器上，其中上下文切换效率高，大多数工作负载在没有对并发线程数量设置任何限制的情况下运行良好。

在需要最小化线程之间上下文切换的情况下，`InnoDB`可以使用多种技术来限制同时执行的操作系统线程数量（因此同时处理的请求数量）。当`InnoDB`从用户会话接收到一个新请求时，如果同时执行的线程数量达到预定义的限制，新请求会在再次尝试之前短暂休眠一段时间。等待锁的线程不计入同时执行的线程数量。

您可以通过设置配置参数`innodb_thread_concurrency`来限制并发线程的数量。一旦执行线程数量达到此限制，额外的线程会在被放入队列之前休眠一段由配置参数`innodb_thread_sleep_delay`设置的微秒数。

您可以将配置选项`innodb_adaptive_max_sleep_delay`设置为您允许的最高值，以及`InnoDB`会根据当前线程调度活动自动调整`innodb_thread_sleep_delay`的值。这种动态调整有助于在线程调度机制在系统轻载时和在接近满负荷运行时平稳工作。

在各个 MySQL 和`InnoDB`的发布版本中，`innodb_thread_concurrency`的默认值和隐含的并发线程数量限制已经发生了变化。`innodb_thread_concurrency`的默认值为`0`，因此默认情况下没有对同时执行的线程数量设置限制。

`InnoDB`只有在并发线程数量有限时才会使线程进入睡眠状态。当线程数量没有限制时，所有线程都会平等竞争调度。也就是说，如果`innodb_thread_concurrency`为`0`，则`innodb_thread_sleep_delay`的值会被忽略。

当线程数量有限时（当`innodb_thread_concurrency` > 0 时），`InnoDB`通过允许在执行*单个 SQL 语句*期间发出的多个请求进入`InnoDB`来减少上下文切换开销，而无需遵守`innodb_thread_concurrency`设置的限制。由于一个 SQL 语句（如连接）可能包含`InnoDB`内的多个行操作，`InnoDB`会分配一定数量的“票据”，允许线程以最小的开销重复调度。

当一个新的 SQL 语句开始执行时，线程没有任何票据，必须遵守`innodb_thread_concurrency`。一旦线程有资格进入`InnoDB`，它将被分配一定数量的票据，用于随后进入`InnoDB`执行行操作。如果票据用完，线程将被驱逐，并且会再次观察`innodb_thread_concurrency`，这可能会将线程重新放入等待线程的先进先出队列中。当线程再次有资格进入`InnoDB`时，将再次分配票据。分配的票据数量由全局选项`innodb_concurrency_tickets`指定，默认值为 5000。等待锁的线程在锁可用时会获得一个票据。

这些变量的正确值取决于您的环境和工作负载。尝试一系列不同的值，以确定哪个值适用于您的应用程序。在限制并发执行线程数量之前，请查看可能改善多核和多处理器计算机上`InnoDB`性能的配置选项，例如`innodb_adaptive_hash_index`。

关于 MySQL 线程处理的一般性性能信息，请参见第 7.1.12.1 节，“连接接口”。
