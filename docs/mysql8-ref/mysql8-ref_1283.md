> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-tuning.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-tuning.html)

#### 17.20.6.3 调整 InnoDB memcached 插件性能

因为使用`InnoDB`与**memcached**结合涉及将所有数据写入磁盘，无论是立即还是稍后，所以原始性能预计会比单独使用**memcached**略慢。在使用`InnoDB`**memcached**插件时，将**memcached**操作的调整目标集中在实现比等效 SQL 操作更好的性能上。

基准测试表明，使用**memcached**接口的查询和 DML 操作（插入、更新和删除）比传统 SQL 更快。DML 操作通常会看到更大的改进。因此，首先考虑调整写入密集型应用程序以使用**memcached**接口。还要考虑优先适应使用快速、轻量级机制但缺乏可靠性的写入密集型应用程序。

##### 调整 SQL 查询

最适合简单`GET`请求的查询类型是具有单个子句或`WHERE`子句中一组`AND`条件的查询：

```sql
SQL:
SELECT col FROM tbl WHERE key = 'key_value';

memcached:
get key_value

SQL:
SELECT col FROM tbl WHERE col1 = val1 and col2 = val2 and col3 = val3;

memcached:
# Since you must always know these 3 values to look up the key,
# combine them into a unique string and use that as the key
# for all ADD, SET, and GET operations.
key_value = val1 + ":" + val2 + ":" + val3
get key_value

SQL:
SELECT 'key exists!' FROM tbl
  WHERE EXISTS (SELECT col1 FROM tbl WHERE KEY = 'key_value') LIMIT 1;

memcached:
# Test for existence of key by asking for its value and checking if the call succeeds,
# ignoring the value itself. For existence checking, you typically only store a very
# short value such as "1".
get key_value
```

##### 利用系统内存

为了获得最佳性能，请在配置为典型数据库服务器的机器上部署`daemon_memcached`插件，其中大部分系统 RAM 专用于`InnoDB`缓冲池，通过`innodb_buffer_pool_size`配置选项。对于具有多千兆字节缓冲池的系统，考虑提高`innodb_buffer_pool_instances`的值，以获得大多数操作涉及已缓存在内存中的数据时的最大吞吐量。

##### 减少冗余 I/O

`InnoDB`有许多设置选项，让您在发生崩溃时可以选择高可靠性，同时在高写入工作负载期间减少 I/O 开销。例如，考虑将`innodb_doublewrite`设置为`0`，将`innodb_flush_log_at_trx_commit`设置为`2`。使用不同的`innodb_flush_method`设置来衡量性能。

若要减少或调整表操作的 I/O 的其他方法，请参阅第 10.5.8 节，“优化 InnoDB 磁盘 I/O”。

##### 减少事务开销

`daemon_memcached_r_batch_size`和`daemon_memcached_w_batch_size`的默认值为 1，旨在获得最大的结果可靠性和存储或更新数据的安全性。

根据应用程序的类型，您可能会增加这两个设置中的一个或两个，以减少频繁提交操作的开销。在繁忙的系统上，您可能会增加`daemon_memcached_r_batch_size`的值，知道通过 SQL 对数据的更改可能不会立即对**memcached**可见（也就是说，直到处理了*`N`*次`get`操作）。在处理每个写操作都必须可靠存储的数据时，将`daemon_memcached_w_batch_size`设置为`1`。在处理仅用于统计分析的大量更新时，可以增加该设置，其中在意外退出时丢失最后*`N`*次更新是可以接受的风险。

例如，想象一个监视穿过繁忙桥梁的交通的系统，每天记录大约 10 万辆车辆的数据。如果应用程序计算不同类型的车辆以分析交通模式，将`daemon_memcached_w_batch_size`从`1`更改为`100`可以将提交操作的 I/O 开销减少 99%。在发生故障时，最多会丢失 100 条记录，这可能是可以接受的误差范围。如果应用程序执行每辆车的自动收费，您将把`daemon_memcached_w_batch_size`设置为`1`，以确保每个收费记录立即保存到磁盘。

由于`InnoDB`在磁盘上组织**memcached**键值的方式，如果要创建大量键，则在应用程序中按键值对数据项进行排序并按排序顺序`add`它们可能比以任意顺序创建键更快。

**memslap**命令是常规**memcached**分发的一部分，但不包括在`daemon_memcached`插件中，可用于对不同配置进行基准测试。它还可用于生成样本键值对，以在您自己的基准测试中使用。
