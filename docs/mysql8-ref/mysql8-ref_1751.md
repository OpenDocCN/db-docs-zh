# 25.6.16 ndbinfo: NDB 集群信息数据库

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo.html)

25.6.16.1 ndbinfo arbitrator_validity_detail 表

25.6.16.2 ndbinfo arbitrator_validity_summary 表

25.6.16.3 ndbinfo backup_id 表

25.6.16.4 ndbinfo blobs 表

25.6.16.5 ndbinfo blocks 表

25.6.16.6 ndbinfo cluster_locks 表

25.6.16.7 ndbinfo cluster_operations 表

25.6.16.8 ndbinfo cluster_transactions 表

25.6.16.9 ndbinfo config_nodes 表

25.6.16.10 ndbinfo config_params 表

25.6.16.11 ndbinfo config_values 表

25.6.16.12 ndbinfo counters 表

25.6.16.13 ndbinfo cpudata 表

25.6.16.14 ndbinfo cpudata_1sec 表

25.6.16.15 ndbinfo cpudata_20sec 表

25.6.16.16 ndbinfo cpudata_50ms 表

25.6.16.17 ndbinfo cpuinfo 表

25.6.16.18 ndbinfo cpustat 表

25.6.16.19 ndbinfo cpustat_50ms 表

25.6.16.20 ndbinfo cpustat_1sec 表

25.6.16.21 ndbinfo cpustat_20sec 表

25.6.16.22 ndbinfo dictionary_columns 表

25.6.16.23 ndbinfo dictionary_tables 表

25.6.16.24 ndbinfo dict_obj_info 表

25.6.16.25 ndbinfo dict_obj_tree 表

25.6.16.26 ndbinfo dict_obj_types 表

25.6.16.27 ndbinfo disk_write_speed_base 表

25.6.16.28 ndbinfo disk_write_speed_aggregate 表

25.6.16.29 ndbinfo disk_write_speed_aggregate_node 表

25.6.16.30 ndbinfo diskpagebuffer 表

25.6.16.31 ndbinfo diskstat 表

25.6.16.32 ndbinfo diskstats_1sec 表

25.6.16.33 ndbinfo error_messages 表

25.6.16.34 ndbinfo events 表

25.6.16.35 ndbinfo files 表

25.6.16.36 ndbinfo foreign_keys 表

25.6.16.37 ndbinfo hash_maps 表

25.6.16.38 ndbinfo hwinfo 表

25.6.16.39 ndbinfo index_columns 表

25.6.16.40 ndbinfo index_stats 表

25.6.16.41 ndbinfo locks_per_fragment 表

25.6.16.42 ndbinfo logbuffers 表

25.6.16.43 ndbinfo logspaces 表

25.6.16.44 ndbinfo membership 表

25.6.16.45 ndbinfo memoryusage 表

25.6.16.46 ndbinfo memory_per_fragment 表

25.6.16.47 ndbinfo nodes 表

25.6.16.48 ndbinfo operations_per_fragment 表

25.6.16.49 ndbinfo pgman_time_track_stats 表

25.6.16.50 ndbinfo processes 表

25.6.16.51 ndbinfo resources 表

25.6.16.52 ndbinfo restart_info 表

25.6.16.53 ndbinfo server_locks 表

25.6.16.54 ndbinfo server_operations 表

25.6.16.55 ndbinfo server_transactions 表

25.6.16.56 ndbinfo table_distribution_status 表

25.6.16.57 ndbinfo table_fragments 表

25.6.16.58 ndbinfo table_info 表

25.6.16.59 ndbinfo table_replicas 表

25.6.16.60 ndbinfo tc_time_track_stats 表

25.6.16.61 ndbinfo threadblocks 表

25.6.16.62 ndbinfo threads 表

25.6.16.63 ndbinfo threadstat 表

25.6.16.64 ndbinfo transporter_details 表

25.6.16.65 ndbinfo transporters 表

`ndbinfo`是一个包含特定于 NDB Cluster 的信息的数据库。

该数据库包含许多表，每个表提供关于 NDB Cluster 节点状态、资源使用情况和操作的不同类型数据。您可以在接下来的几节中找到关于每个表的更详细信息。

`ndbinfo`包含在 MySQL Server 中的 NDB Cluster 支持中；不需要特殊的编译或配置步骤；当 MySQL Server 连接到集群时，这些表由 MySQL Server 创建。您可以使用`SHOW PLUGINS`验证给定 MySQL Server 实例中是否激活了`ndbinfo`支持；如果启用了`ndbinfo`支持，您应该看到`Name`列中包含`ndbinfo`，`Status`列中包含`ACTIVE`的行，如下所示（强调文本）：

```sql
mysql> SHOW PLUGINS;
+----------------------------------+--------+--------------------+---------+---------+
| Name                             | Status | Type               | Library | License |
+----------------------------------+--------+--------------------+---------+---------+
| binlog                           | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| mysql_native_password            | ACTIVE | AUTHENTICATION     | NULL    | GPL     |
| sha256_password                  | ACTIVE | AUTHENTICATION     | NULL    | GPL     |
| caching_sha2_password            | ACTIVE | AUTHENTICATION     | NULL    | GPL     |
| sha2_cache_cleaner               | ACTIVE | AUDIT              | NULL    | GPL     |
| daemon_keyring_proxy_plugin      | ACTIVE | DAEMON             | NULL    | GPL     |
| CSV                              | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| MEMORY                           | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| InnoDB                           | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| INNODB_TRX                       | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_CMP                       | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_CMP_RESET                 | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_CMPMEM                    | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_CMPMEM_RESET              | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_CMP_PER_INDEX             | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_CMP_PER_INDEX_RESET       | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_BUFFER_PAGE               | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_BUFFER_PAGE_LRU           | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_BUFFER_POOL_STATS         | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_TEMP_TABLE_INFO           | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_METRICS                   | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_FT_DEFAULT_STOPWORD       | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_FT_DELETED                | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_FT_BEING_DELETED          | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_FT_CONFIG                 | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_FT_INDEX_CACHE            | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_FT_INDEX_TABLE            | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_TABLES                    | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_TABLESTATS                | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_INDEXES                   | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_TABLESPACES               | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_COLUMNS                   | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_VIRTUAL                   | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_CACHED_INDEXES            | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_SESSION_TEMP_TABLESPACES  | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| MyISAM                           | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| MRG_MYISAM                       | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| PERFORMANCE_SCHEMA               | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| TempTable                        | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| ARCHIVE                          | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| BLACKHOLE                        | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| ndbcluster                       | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
*| ndbinfo                          | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |*
| ndb_transid_mysql_connection_map | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| ngram                            | ACTIVE | FTPARSER           | NULL    | GPL     |
| mysqlx_cache_cleaner             | ACTIVE | AUDIT              | NULL    | GPL     |
| mysqlx                           | ACTIVE | DAEMON             | NULL    | GPL     |
+----------------------------------+--------+--------------------+---------+---------+
47 rows in set (0.00 sec)
```

您还可以通过检查`SHOW ENGINES`的输出，查看包含`ndbinfo`的行以及`Support`列中的`YES`，如下所示（强调文本）：

```sql
mysql> SHOW ENGINES\G
*************************** 1\. row ***************************
      Engine: ndbcluster
     Support: YES
     Comment: Clustered, fault-tolerant tables
Transactions: YES
          XA: NO
  Savepoints: NO
*************************** 2\. row ***************************
      Engine: CSV
     Support: YES
     Comment: CSV storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 3\. row ***************************
      Engine: InnoDB
     Support: DEFAULT
     Comment: Supports transactions, row-level locking, and foreign keys
Transactions: YES
          XA: YES
  Savepoints: YES
*************************** 4\. row ***************************
      Engine: BLACKHOLE
     Support: YES
     Comment: /dev/null storage engine (anything you write to it disappears)
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 5\. row ***************************
      Engine: MyISAM
     Support: YES
     Comment: MyISAM storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 6\. row ***************************
      Engine: MRG_MYISAM
     Support: YES
     Comment: Collection of identical MyISAM tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 7\. row ***************************
      Engine: ARCHIVE
     Support: YES
     Comment: Archive storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
**************************** 8\. row ***************************
      Engine: ndbinfo
     Support: YES
     Comment: NDB Cluster system information storage engine
Transactions: NO
          XA: NO
  Savepoints: NO*
*************************** 9\. row ***************************
      Engine: PERFORMANCE_SCHEMA
     Support: YES
     Comment: Performance Schema
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 10\. row ***************************
      Engine: MEMORY
     Support: YES
     Comment: Hash based, stored in memory, useful for temporary tables
Transactions: NO
          XA: NO
  Savepoints: NO 10 rows in set (0.00 sec)
```

如果启用了`ndbinfo`支持，则可以使用 SQL 语句在**mysql**或其他 MySQL 客户端中访问`ndbinfo`。例如，您可以在`SHOW DATABASES`的输出中看到`ndbinfo`，如下所示（强调文本）：

```sql
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
*| ndbinfo            |*
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.04 sec)
```

如果**mysqld**进程未使用`--ndbcluster`选项启动，则`ndbinfo`不可用，并且不会显示在`SHOW DATABASES`中。如果**mysqld**曾连接到 NDB Cluster 但集群不可用（由于事件如集群关闭、网络连接丢失等），`ndbinfo`及其表仍然可见，但尝试访问任何表（除了`blocks`或`config_params`）将失败，并显示错误 157 'Connection to NDB failed' from NDBINFO。

除了`blocks`和`config_params`表之外，我们所说的`ndbinfo`“表”实际上是从内部`NDB`表生成的视图，通常对 MySQL Server 不可见。您可以通过将`ndbinfo_show_hidden`系统变量设置为`ON`（或`1`）来使这些表可见，但通常不需要这样做。

所有 `ndbinfo` 表都是只读的，并在查询时按需生成。因为许多表是由数据节点并行生成的，而其他表是特定于给定的 SQL 节点，所以不能保证提供一致的快照。

此外，在 `ndbinfo` 表上不支持连接的下推；因此，连接大型 `ndbinfo` 表可能需要将大量数据传输到请求的 API 节点，即使查询使用 `WHERE` 子句。

`ndbinfo` 表不包含在查询缓存中。（Bug #59831）

你可以使用 `USE` 语句选择 `ndbinfo` 数据库，然后发出 `SHOW TABLES` 语句获取表的列表，就像对待任何其他数据库一样，如下所示：

```sql
mysql> USE ndbinfo;
Database changed

mysql> SHOW TABLES;
+---------------------------------+
| Tables_in_ndbinfo               |
+---------------------------------+
| arbitrator_validity_detail      |
| arbitrator_validity_summary     |
| backup_id                       |
| blobs                           |
| blocks                          |
| cluster_locks                   |
| cluster_operations              |
| cluster_transactions            |
| config_nodes                    |
| config_params                   |
| config_values                   |
| counters                        |
| cpudata                         |
| cpudata_1sec                    |
| cpudata_20sec                   |
| cpudata_50ms                    |
| cpuinfo                         |
| cpustat                         |
| cpustat_1sec                    |
| cpustat_20sec                   |
| cpustat_50ms                    |
| dict_obj_info                   |
| dict_obj_tree                   |
| dict_obj_types                  |
| dictionary_columns              |
| dictionary_tables               |
| disk_write_speed_aggregate      |
| disk_write_speed_aggregate_node |
| disk_write_speed_base           |
| diskpagebuffer                  |
| diskstat                        |
| diskstats_1sec                  |
| error_messages                  |
| events                          |
| files                           |
| foreign_keys                    |
| hash_maps                       |
| hwinfo                          |
| index_columns                   |
| index_stats                     |
| locks_per_fragment              |
| logbuffers                      |
| logspaces                       |
| membership                      |
| memory_per_fragment             |
| memoryusage                     |
| nodes                           |
| operations_per_fragment         |
| pgman_time_track_stats          |
| processes                       |
| resources                       |
| restart_info                    |
| server_locks                    |
| server_operations               |
| server_transactions             |
| table_distribution_status       |
| table_fragments                 |
| table_info                      |
| table_replicas                  |
| tc_time_track_stats             |
| threadblocks                    |
| threads                         |
| threadstat                      |
| transporters                    |
+---------------------------------+
64 rows in set (0.00 sec)
```

在 NDB 8.0 中，所有 `ndbinfo` 表都使用 `NDB` 存储引擎；然而，在`SHOW ENGINES`和`SHOW PLUGINS`的输出中仍会出现一个 `ndbinfo` 条目，如前所述。

你可以执行对这些表的`SELECT`语句，就像你通常期望的那样：

```sql
mysql> SELECT * FROM memoryusage;
+---------+---------------------+--------+------------+------------+-------------+
| node_id | memory_type         | used   | used_pages | total      | total_pages |
+---------+---------------------+--------+------------+------------+-------------+
|       5 | Data memory         | 425984 |         13 | 2147483648 |       65536 |
|       5 | Long message buffer | 393216 |       1536 |   67108864 |      262144 |
|       6 | Data memory         | 425984 |         13 | 2147483648 |       65536 |
|       6 | Long message buffer | 393216 |       1536 |   67108864 |      262144 |
|       7 | Data memory         | 425984 |         13 | 2147483648 |       65536 |
|       7 | Long message buffer | 393216 |       1536 |   67108864 |      262144 |
|       8 | Data memory         | 425984 |         13 | 2147483648 |       65536 |
|       8 | Long message buffer | 393216 |       1536 |   67108864 |      262144 |
+---------+---------------------+--------+------------+------------+-------------+
8 rows in set (0.09 sec)
```

更复杂的查询，比如以下两个使用`memoryusage`表的`SELECT`语句，是可能的：

```sql
mysql> SELECT SUM(used) as 'Data Memory Used, All Nodes'
     >     FROM memoryusage
     >     WHERE memory_type = 'Data memory';
+-----------------------------+
| Data Memory Used, All Nodes |
+-----------------------------+
|                        6460 |
+-----------------------------+
1 row in set (0.09 sec)

mysql> SELECT SUM(used) as 'Long Message Buffer, All Nodes'
     >     FROM memoryusage
     >     WHERE memory_type = 'Long message buffer';
+-------------------------------------+
| Long Message Buffer Used, All Nodes |
+-------------------------------------+
|                             1179648 |
+-------------------------------------+
1 row in set (0.08 sec)
```

`ndbinfo` 表和列名区分大小写（`ndbinfo` 数据库本身的名称也是如此）。这些标识符是小写的。尝试使用错误的大小写会导致错误，如下例所示：

```sql
mysql> SELECT * FROM nodes;
+---------+--------+---------+-------------+-------------------+
| node_id | uptime | status  | start_phase | config_generation |
+---------+--------+---------+-------------+-------------------+
|       5 |  17707 | STARTED |           0 |                 1 |
|       6 |  17706 | STARTED |           0 |                 1 |
|       7 |  17705 | STARTED |           0 |                 1 |
|       8 |  17704 | STARTED |           0 |                 1 |
+---------+--------+---------+-------------+-------------------+
4 rows in set (0.06 sec)

mysql> SELECT * FROM Nodes;
ERROR 1146 (42S02): Table 'ndbinfo.Nodes' doesn't exist
```

**mysqldump** 完全忽略 `ndbinfo` 数据库，并将其从任何输出中排除。即使使用 `--databases` 或 `--all-databases` 选项也是如此。

NDB Cluster 还在 `INFORMATION_SCHEMA` 信息数据库中维护表，包括包含有关用于 NDB Cluster 磁盘数据存储的文件的信息的`FILES`表，以及显示事务、事务协调器和 NDB Cluster API 节点之间关系的`ndb_transid_mysql_connection_map`表。有关更多信息，请参阅表的描述或第 25.6.17 节，“NDB Cluster 的 INFORMATION_SCHEMA 表”。
