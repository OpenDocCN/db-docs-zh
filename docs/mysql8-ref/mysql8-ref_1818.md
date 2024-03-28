# 25.6.18 NDB Cluster and the Performance Schema

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ps-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ps-tables.html)

NDB 8.0 提供了关于线程和事务内存使用情况的 MySQL 性能模式信息；NDB 8.0.29 添加了`ndbcluster`插件线程，NDB 8.0.30 添加了事务批处理内存的仪表化。这些功能在接下来的章节中有更详细的描述。

#### ndbcluster 插件线程

从 NDB 8.0.29 开始，`ndbcluster`插件线程在性能模式`threads`表中可见，如下查询所示：

```sql
mysql> SELECT name, type, thread_id, thread_os_id
 -> FROM performance_schema.threads
 -> WHERE name LIKE '%ndbcluster%'\G
+----------------------------------+------------+-----------+--------------+
| name                             | type       | thread_id | thread_os_id |
+----------------------------------+------------+-----------+--------------+
| thread/ndbcluster/ndb_binlog     | BACKGROUND |        30 |        11980 |
| thread/ndbcluster/ndb_index_stat | BACKGROUND |        31 |        11981 |
| thread/ndbcluster/ndb_metadata   | BACKGROUND |        32 |        11982 |
+----------------------------------+------------+-----------+--------------+
```

`threads`表显示了这里列出的所有三个线程：

+   `ndb_binlog`：二进制日志线程

+   `ndb_index_stat`：索引统计线程

+   `ndb_metadata`：元数据线程

这些线程也在`setup_threads`表中按名称显示。

线程名称在`threads`和`setup_threads`表的`name`列中以`*`prefix`*/*`plugin_name`*/*`thread_name`*`的格式显示。 *`prefix`*是由`performance_schema`引擎确定的对象类型，对于插件线程是`thread`（参见 Thread Instrument Elements）。 *`plugin_name`*是`ndbcluster`。 *`thread_name`*是线程的独立名称（`ndb_binlog`、`ndb_index_stat`或`ndb_metadata`）。

使用`threads`或`setup_threads`表中给定线程的线程 ID 或 OS 线程 ID，可以从性能模式中获取有关插件执行和资源使用的大量信息。此示例显示了如何通过连接`threads`和`memory_summary_by_thread_by_event_name`表来获取`ndbcluster`插件创建的线程在`mem_root`区域分配的内存量：

```sql
mysql> SELECT
 ->   t.name,
 ->   m.sum_number_of_bytes_alloc,
 ->   IF(m.sum_number_of_bytes_alloc > 0, "true", "false") AS 'Has allocated memory'
 -> FROM performance_schema.memory_summary_by_thread_by_event_name m
 -> JOIN performance_schema.threads t
 -> ON m.thread_id = t.thread_id
 -> WHERE t.name LIKE '%ndbcluster%'
 ->   AND event_name LIKE '%THD::main_mem_root%';
+----------------------------------+---------------------------+----------------------+
| name                             | sum_number_of_bytes_alloc | Has allocated memory |
+----------------------------------+---------------------------+----------------------+
| thread/ndbcluster/ndb_binlog     |                     20576 | true                 |
| thread/ndbcluster/ndb_index_stat |                         0 | false                |
| thread/ndbcluster/ndb_metadata   |                      8240 | true                 |
+----------------------------------+---------------------------+----------------------+
```

#### 事务内存使用情况

从 NDB 8.0.30 开始，您可以通过查询性能模式`memory_summary_by_thread_by_event_name`表来查看事务批处理使用的内存量，类似于下面显示的内容：

```sql
mysql> SELECT EVENT_NAME
 ->   FROM performance_schema.memory_summary_by_thread_by_event_name
 ->   WHERE THREAD_ID = PS_CURRENT_THREAD_ID()
 ->     AND EVENT_NAME LIKE 'memory/ndbcluster/%';
+-------------------------------------------+
| EVENT_NAME                                |
+-------------------------------------------+
| memory/ndbcluster/Thd_ndb::batch_mem_root |
+-------------------------------------------+
1 row in set (0.01 sec)
```

`ndbcluster`事务内存仪器也在性能模式`setup_instruments`表中可见，如下所示：

```sql
mysql> SELECT * from performance_schema.setup_instruments 
 ->   WHERE NAME LIKE '%ndb%'\G
*************************** 1\. row ***************************
         NAME: memory/ndbcluster/Thd_ndb::batch_mem_root
      ENABLED: YES
        TIMED: NULL
   PROPERTIES: 
   VOLATILITY: 0
DOCUMENTATION: Memory used for transaction batching 1 row in set (0.01 sec)
```
