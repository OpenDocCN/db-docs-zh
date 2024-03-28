> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-multiple-buffer-pools.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-multiple-buffer-pools.html)

#### 17.8.3.2 配置多个缓冲池实例

对于具有多吉字节范围缓冲池的系统，将缓冲池划分为单独的实例可以提高并发性，通过减少不同线程读取和写入缓存页面时的争用。此功能通常适用于具有多吉字节范围缓冲池大小的系统。可以使用`innodb_buffer_pool_instances`配置选项配置多个缓冲池实例，并且您可能还需要调整`innodb_buffer_pool_size`的值。

当`InnoDB`缓冲池很大时，许多数据请求可以通过从内存中检索来满足。您可能会遇到多个线程同时尝试访问缓冲池而导致瓶颈。您可以启用多个缓冲池以最小化此争用。存储在缓冲池中或从缓冲池中读取的每个页面都随机分配给其中一个缓冲池，使用哈希函数。每个缓冲池管理其自己的空闲列表、刷新列表、LRU 列表和所有与缓冲池相关的其他数据结构。在 MySQL 8.0 之前，每个缓冲池都由其自己的缓冲池互斥锁保护。在 MySQL 8.0 及更高版本中，缓冲池互斥锁被几个列表和哈希保护互斥锁所取代，以减少争用。

要启用多个缓冲池实例，请将`innodb_buffer_pool_instances`配置选项设置为大于 1（默认值）至 64（最大值）。仅当您将`innodb_buffer_pool_size`设置为 1GB 或更大时，此选项才会生效。您指定的总大小将分配给所有缓冲池。为了获得最佳效率，请指定`innodb_buffer_pool_instances`和`innodb_buffer_pool_size`的组合，以便每个缓冲池实例至少为 1GB。

要了解如何修改`InnoDB`缓冲池大小的信息，请参阅第 17.8.3.1 节，“配置 InnoDB 缓冲池大小”。
