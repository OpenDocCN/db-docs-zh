> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-memoryusage.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-memoryusage.html)

#### 25.6.16.45 `ndbinfo memoryusage`表

查询此表提供类似于`ALL REPORT MemoryUsage`命令在**ndb_mgm**客户端中提供的信息，或由`ALL DUMP 1000`记录的信息。

`memoryusage`表包含以下列：

+   `node_id`

    此数据节点的节点 ID。

+   `memory_type`

    `Data memory`、`Index memory`或`Long message buffer`中的一个。

+   `used`

    此数据节点当前用于数据内存或索引内存的字节数。

+   `used_pages`

    此数据节点当前用于数据内存或索引内存的页面数；参见文本。

+   `total`

    数据节点可用的数据内存或索引内存的总字节数；参见文本。

+   `total_pages`

    数据内存或索引内存在此数据节点上可用的内存页总数；参见文本。

##### 注

`total`列表示特定数据节点上给定资源（数据内存或索引内存）的可用内存总量（以字节为单位）。此数字应大致等于`config.ini`文件中相应配置参数的设置。

假设集群有 2 个数据节点，节点 ID 分别为`5`和`6`，`config.ini`文件包含以下内容：

```sql
[ndbd default]
DataMemory = 1G
IndexMemory = 1G
```

还假设`LongMessageBuffer`配置参数的值允许采用其默认值（64 MB）。

以下查询显示大致相同的值：

```sql
mysql> SELECT node_id, memory_type, total
     > FROM ndbinfo.memoryusage;
+---------+---------------------+------------+
| node_id | memory_type         | total      |
+---------+---------------------+------------+
|       5 | Data memory         | 1073741824 |
|       5 | Index memory        | 1074003968 |
|       5 | Long message buffer |   67108864 |
|       6 | Data memory         | 1073741824 |
|       6 | Index memory        | 1074003968 |
|       6 | Long message buffer |   67108864 |
+---------+---------------------+------------+
6 rows in set (0.00 sec)
```

在这种情况下，索引内存的`total`列值略高于`IndexMemory`设置值，这是由于内部四舍五入导致的。

对于`used_pages`和`total_pages`列，资源以页面为单位进行度量，对于`DataMemory`为 32K，对于`IndexMemory`为 8K。对于长消息缓冲内存，页面大小为 256 字节。
