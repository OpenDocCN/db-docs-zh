> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-diskpagebuffer.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-diskpagebuffer.html)

#### 25.6.16.30 ndbinfo diskpagebuffer 表

`diskpagebuffer` 表提供有关 NDB Cluster Disk Data 表使用的磁盘页面缓冲区的统计信息。

`diskpagebuffer` 表包含以下列：

+   `node_id`

    数据节点 ID

+   `block_instance`

    块实例

+   `pages_written`

    写入磁盘的页面数量。

+   `pages_written_lcp`

    本地检查点写入的页面数量。

+   `pages_read`

    从磁盘读取的页面数量

+   `log_waits`

    等待将页面写入磁盘的页面写入数量

+   `page_requests_direct_return`

    请求页面在缓冲区中可用的请求数量

+   `page_requests_wait_queue`

    等待页面在缓冲区中变为可用的请求数量

+   `page_requests_wait_io`

    需要从磁盘上的页面读取的请求数量（页面在缓冲区中不可用）

##### 注意

您可以将此表与 NDB Cluster Disk Data 表一起使用，以确定 `DiskPageBufferMemory` 是否足够大，以允许数据从缓冲区而不是磁盘读取；减少磁盘寻道可以帮助提高这些表的性能。

您可以使用类似以下查询来确定从 `DiskPageBufferMemory` 读取的比例占总读取次数的百分比，从而获得这个比率：

```sql
SELECT
  node_id,
  100 * page_requests_direct_return /
    (page_requests_direct_return + page_requests_wait_io)
      AS hit_ratio
FROM ndbinfo.diskpagebuffer;
```

此查询的结果应类似于此处显示的内容，每个数据节点在集群中占据一行（在此示例中，集群有 4 个数据节点）：

```sql
+---------+-----------+
| node_id | hit_ratio |
+---------+-----------+
|       5 |   97.6744 |
|       6 |   97.6879 |
|       7 |   98.1776 |
|       8 |   98.1343 |
+---------+-----------+
4 rows in set (0.00 sec)
```

`hit_ratio` 值接近 100% 表明只有极少数读取是从磁盘而不是缓冲区进行的，这意味着磁盘数据读取性能接近最佳水平。如果其中任何值低于 95%，这是一个强烈的指示，需要在 `config.ini` 文件中增加 `DiskPageBufferMemory` 的设置。

注意

更改 `DiskPageBufferMemory` 需要在其生效之前对集群的所有数据节点进行滚动重启。

`block_instance` 指的是内核块的一个实例。与块名称一起，此数字可用于在 `threadblocks` 表中查找给定实例。使用这些信息，您可以获取与单个线程相关的磁盘页面缓冲区指标的信息；这里显示了一个使用 `LIMIT 1` 限制输出为单个线程的示例查询：

```sql
mysql> SELECT
     >   node_id, thr_no, block_name, thread_name, pages_written,
     >   pages_written_lcp, pages_read, log_waits,
     >   page_requests_direct_return, page_requests_wait_queue,
     >   page_requests_wait_io
     > FROM ndbinfo.diskpagebuffer
     >   INNER JOIN ndbinfo.threadblocks USING (node_id, block_instance)
     >   INNER JOIN ndbinfo.threads USING (node_id, thr_no)
     > WHERE block_name = 'PGMAN' LIMIT 1\G
*************************** 1\. row ***************************
                    node_id: 1
                     thr_no: 1
                 block_name: PGMAN
                thread_name: rep
              pages_written: 0
          pages_written_lcp: 0
                 pages_read: 1
                  log_waits: 0
page_requests_direct_return: 4
   page_requests_wait_queue: 0
      page_requests_wait_io: 1 1 row in set (0.01 sec)
```
