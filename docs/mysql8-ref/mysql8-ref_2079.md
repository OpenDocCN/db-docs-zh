# 29.12.16 性能模式线程池表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-thread-pool-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-thread-pool-tables.html)

29.12.16.1 线程组状态表

29.12.16.2 线程组统计表

29.12.16.3 线程状态表

注意

此处描述的性能模式表自 MySQL 8.0.14 版本起可用。在 MySQL 8.0.14 之前，请改用相应的`INFORMATION_SCHEMA`表；请参阅第 28.5 节，“INFORMATION_SCHEMA 线程池表”。

以下部分描述了与线程池插件相关的性能模式表（参见第 7.6.3 节，“MySQL 企业线程池”）。它们提供有关线程池操作的信息：

+   `tp_thread_group_state`：关于线程池线程组状态的信息。

+   `tp_thread_group_stats`：线程组统计信息。

+   `tp_thread_state`：关于线程池线程状态的信息。

这些表中的行代表时间的快照。在`tp_thread_state`的情况下，线程组的所有行组成一个时间快照。因此，MySQL 服务器在生成快照时持有线程组的互斥锁。但它不会同时持有所有线程组的互斥锁，以防止针对`tp_thread_state`的语句阻塞整个 MySQL 服务器。

性能模式线程池表由线程池插件实现，并在加载和卸载该插件时加载和卸载（参见第 7.6.3.2 节，“线程池安装”）。表不需要特殊配置步骤。但是，表依赖于启用线程池插件。如果加载了线程池插件但未启用，则不会创建表。
