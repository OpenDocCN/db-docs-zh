# 28.5 INFORMATION_SCHEMA 线程池表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/thread-pool-information-schema-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/thread-pool-information-schema-tables.html)

28.5.1 INFORMATION_SCHEMA 线程池表参考

28.5.2 INFORMATION_SCHEMA TP_THREAD_GROUP_STATE 表

28.5.3 INFORMATION_SCHEMA TP_THREAD_GROUP_STATS 表

28.5.4 INFORMATION_SCHEMA TP_THREAD_STATE 表

注意

截至 MySQL 8.0.14，`INFORMATION_SCHEMA`线程池表也可作为性能模式表使用（参见第 29.12.16 节，“性能模式线程池表”：关于线程池线程组状态的信息

+   `TP_THREAD_GROUP_STATS`：线程组统计信息

+   `TP_THREAD_STATE`：关于线程池线程状态的信息

这些表中的行代表了某个时间点的快照。在`TP_THREAD_STATE`的情况下，线程组的所有行构成了一个时间点的快照。因此，MySQL 服务器在生成快照时持有线程组的互斥锁。但它不会同时持有所有线程组的互斥锁，以防止针对`TP_THREAD_STATE`的语句阻塞整个 MySQL 服务器。

`INFORMATION_SCHEMA`线程池表由各个插件实现，是否加载一个插件的决定可以独立于其他插件（参见[第 7.6.3.2 节，“线程池安装”](thread-pool-installation.html "7.6.3.2 Thread Pool Installation"））。但是，所有表的内容取决于启用线程池插件。如果启用了表插件但未启用线程池插件，则该表变为可见并且可以访问，但为空。
