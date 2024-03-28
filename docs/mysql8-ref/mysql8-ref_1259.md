# 17.15.5 InnoDB INFORMATION_SCHEMA 缓冲池表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-buffer-pool-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-buffer-pool-tables.html)

`InnoDB` `INFORMATION_SCHEMA`缓冲池表提供有关`InnoDB`缓冲池内页面的缓冲池状态信息和元数据。

`InnoDB` `INFORMATION_SCHEMA`缓冲池表包括以下列出的表：

```sql
mysql> SHOW TABLES FROM INFORMATION_SCHEMA LIKE 'INNODB_BUFFER%';
+-----------------------------------------------+
| Tables_in_INFORMATION_SCHEMA (INNODB_BUFFER%) |
+-----------------------------------------------+
| INNODB_BUFFER_PAGE_LRU                        |
| INNODB_BUFFER_PAGE                            |
| INNODB_BUFFER_POOL_STATS                      |
+-----------------------------------------------+
```

#### 表概述

+   `INNODB_BUFFER_PAGE`：包含`InnoDB`缓冲池中每个页面的信息。

+   `INNODB_BUFFER_PAGE_LRU`：包含关于`InnoDB`缓冲池中页面的信息，特别是它们在 LRU 列表中的排序方式，该列表确定在缓冲池变满时要驱逐的页面。`INNODB_BUFFER_PAGE_LRU`表与`INNODB_BUFFER_PAGE`表具有相同的列，只是`INNODB_BUFFER_PAGE_LRU`表具有一个`LRU_POSITION`列，而不是一个`BLOCK_ID`列。

+   `INNODB_BUFFER_POOL_STATS`：提供缓冲池状态信息。`SHOW ENGINE INNODB STATUS`输出提供了大部分相同的信息，或者可以使用`InnoDB`缓冲池服务器状态变量获得。

警告

在生产系统上查询`INNODB_BUFFER_PAGE`或`INNODB_BUFFER_PAGE_LRU`表可能会影响性能。除非您意识到性能影响并确定其可接受，否则不要在生产系统上查询这些表。为避免影响生产系统的性能，请在测试实例上重现您想要调查的问题，并查询缓冲池统计信息。

**示例 17.6 在 INNODB_BUFFER_PAGE 表中查询系统数据**

此查询通过排除`TABLE_NAME`值为`NULL`或包含斜杠`/`或句点`.`的页面，提供包含系统数据的页面的近似计数，这表明用户定义了表。

```sql
mysql> SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
       WHERE TABLE_NAME IS NULL OR (INSTR(TABLE_NAME, '/') = 0 AND INSTR(TABLE_NAME, '.') = 0);
+----------+
| COUNT(*) |
+----------+
|     1516 |
+----------+
```

此查询返回包含系统数据的页面的大致数量，缓冲池页面的总��，以及包含系统数据的页面的大致百分比。

```sql
mysql> SELECT
       (SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
       WHERE TABLE_NAME IS NULL OR (INSTR(TABLE_NAME, '/') = 0 AND INSTR(TABLE_NAME, '.') = 0)
       ) AS system_pages,
       (
       SELECT COUNT(*)
       FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
       ) AS total_pages,
       (
       SELECT ROUND((system_pages/total_pages) * 100)
       ) AS system_page_percentage;
+--------------+-------------+------------------------+
| system_pages | total_pages | system_page_percentage |
+--------------+-------------+------------------------+
|          295 |        8192 |                      4 |
+--------------+-------------+------------------------+
```

通过查询 `PAGE_TYPE` 值可以确定缓冲池中的系统数据类型。例如，以下查询返回包含系统数据的页面中的八个不同的 `PAGE_TYPE` 值：

```sql
mysql> SELECT DISTINCT PAGE_TYPE FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
       WHERE TABLE_NAME IS NULL OR (INSTR(TABLE_NAME, '/') = 0 AND INSTR(TABLE_NAME, '.') = 0);
+-------------------+
| PAGE_TYPE         |
+-------------------+
| SYSTEM            |
| IBUF_BITMAP       |
| UNKNOWN           |
| FILE_SPACE_HEADER |
| INODE             |
| UNDO_LOG          |
| ALLOCATED         |
+-------------------+
```

**示例 17.7 在 INNODB_BUFFER_PAGE 表中查询用户数据**

此查询通过计算 `TABLE_NAME` 值为 `NOT NULL` 且 `NOT LIKE '%INNODB_TABLES%'` 的页面数量，提供包含用户数据的页面的大致计数。

```sql
mysql> SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
       WHERE TABLE_NAME IS NOT NULL AND TABLE_NAME NOT LIKE '%INNODB_TABLES%';
+----------+
| COUNT(*) |
+----------+
|     7897 |
+----------+
```

此查询返回包含用户数据的页面的大致数量，缓冲池页面的总数，以及包含用户数据的页面的大致百分比。

```sql
mysql> SELECT
       (SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
       WHERE TABLE_NAME IS NOT NULL AND (INSTR(TABLE_NAME, '/') > 0 OR INSTR(TABLE_NAME, '.') > 0)
       ) AS user_pages,
       (
       SELECT COUNT(*)
       FROM information_schema.INNODB_BUFFER_PAGE
       ) AS total_pages,
       (
       SELECT ROUND((user_pages/total_pages) * 100)
       ) AS user_page_percentage;
+------------+-------------+----------------------+
| user_pages | total_pages | user_page_percentage |
+------------+-------------+----------------------+
|       7897 |        8192 |                   96 |
+------------+-------------+----------------------+
```

此查询标识具有缓冲池中页面的用户定义表：

```sql
mysql> SELECT DISTINCT TABLE_NAME FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
       WHERE TABLE_NAME IS NOT NULL AND (INSTR(TABLE_NAME, '/') > 0 OR INSTR(TABLE_NAME, '.') > 0)
       AND TABLE_NAME NOT LIKE '`mysql`.`innodb_%';
+-------------------------+
| TABLE_NAME              |
+-------------------------+
| `employees`.`salaries`  |
| `employees`.`employees` |
+-------------------------+
```

**示例 17.8 在 INNODB_BUFFER_PAGE 表中查询索引数据**

要了解索引页面的信息，请使用索引的名称查询 `INDEX_NAME` 列。例如，以下查询返回了在 `employees.salaries` 表上定义的 `emp_no` 索引的页面数量和页面的总数据大小：

```sql
mysql> SELECT INDEX_NAME, COUNT(*) AS Pages,
ROUND(SUM(IF(COMPRESSED_SIZE = 0, @@GLOBAL.innodb_page_size, COMPRESSED_SIZE))/1024/1024)
AS 'Total Data (MB)'
FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
WHERE INDEX_NAME='emp_no' AND TABLE_NAME = '`employees`.`salaries`';
+------------+-------+-----------------+
| INDEX_NAME | Pages | Total Data (MB) |
+------------+-------+-----------------+
| emp_no     |  1609 |              25 |
+------------+-------+-----------------+
```

此查询返回了在 `employees.salaries` 表上定义的所有索引的页面数量和页面的总数据大小：

```sql
mysql> SELECT INDEX_NAME, COUNT(*) AS Pages,
       ROUND(SUM(IF(COMPRESSED_SIZE = 0, @@GLOBAL.innodb_page_size, COMPRESSED_SIZE))/1024/1024)
       AS 'Total Data (MB)'
       FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
       WHERE TABLE_NAME = '`employees`.`salaries`'
       GROUP BY INDEX_NAME;
+------------+-------+-----------------+
| INDEX_NAME | Pages | Total Data (MB) |
+------------+-------+-----------------+
| emp_no     |  1608 |              25 |
| PRIMARY    |  6086 |              95 |
+------------+-------+-----------------+
```

**示例 17.9 在 INNODB_BUFFER_PAGE_LRU 表中查询 LRU_POSITION 数据**

`INNODB_BUFFER_PAGE_LRU` 表包含关于 `InnoDB` 缓冲池中页面的信息，特别是它们的排序方式，确定了在缓冲池变满时应该驱逐哪些页面。该页面的定义与 `INNODB_BUFFER_PAGE` 相同，只是该表具有一个 `LRU_POSITION` 列而不是一个 `BLOCK_ID` 列。

此查询计算了 `employees.employees` 表页面在 LRU 列中特定位置上所占的位置数。

```sql
mysql> SELECT COUNT(LRU_POSITION) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE_LRU
       WHERE TABLE_NAME='`employees`.`employees`' AND LRU_POSITION < 3072;
+---------------------+
| COUNT(LRU_POSITION) |
+---------------------+
|                 548 |
+---------------------+
```

**示例 17.10 查询 INNODB_BUFFER_POOL_STATS 表**

`INNODB_BUFFER_POOL_STATS` 表提供类似于 `SHOW ENGINE INNODB STATUS` 和 `InnoDB` 缓冲池状态变量的信息。

```sql
mysql> SELECT * FROM information_schema.INNODB_BUFFER_POOL_STATS \G
*************************** 1\. row ***************************
                         POOL_ID: 0
                       POOL_SIZE: 8192
                    FREE_BUFFERS: 1
                  DATABASE_PAGES: 8173
              OLD_DATABASE_PAGES: 3014
         MODIFIED_DATABASE_PAGES: 0
              PENDING_DECOMPRESS: 0
                   PENDING_READS: 0
               PENDING_FLUSH_LRU: 0
              PENDING_FLUSH_LIST: 0
                PAGES_MADE_YOUNG: 15907
            PAGES_NOT_MADE_YOUNG: 3803101
           PAGES_MADE_YOUNG_RATE: 0
       PAGES_MADE_NOT_YOUNG_RATE: 0
               NUMBER_PAGES_READ: 3270
            NUMBER_PAGES_CREATED: 13176
            NUMBER_PAGES_WRITTEN: 15109
                 PAGES_READ_RATE: 0
               PAGES_CREATE_RATE: 0
              PAGES_WRITTEN_RATE: 0
                NUMBER_PAGES_GET: 33069332
                        HIT_RATE: 0
    YOUNG_MAKE_PER_THOUSAND_GETS: 0
NOT_YOUNG_MAKE_PER_THOUSAND_GETS: 0
         NUMBER_PAGES_READ_AHEAD: 2713
       NUMBER_READ_AHEAD_EVICTED: 0
                 READ_AHEAD_RATE: 0
         READ_AHEAD_EVICTED_RATE: 0
                    LRU_IO_TOTAL: 0
                  LRU_IO_CURRENT: 0
                UNCOMPRESS_TOTAL: 0
              UNCOMPRESS_CURRENT: 0
```

为了比较起见，以下是基于相同数据集的 `SHOW ENGINE INNODB STATUS` 输出和 `InnoDB` 缓冲池状态变量输出。

要了解有关 `SHOW ENGINE INNODB STATUS` 输出的更多信息，请参阅 第 17.17.3 节，“InnoDB 标准监视器和锁监视器输出”。

```sql
mysql> SHOW ENGINE INNODB STATUS \G
...
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 137428992
Dictionary memory allocated 579084
Buffer pool size   8192
Free buffers       1
Database pages     8173
Old database pages 3014
Modified db pages  0
Pending reads 0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 15907, not young 3803101
0.00 youngs/s, 0.00 non-youngs/s
Pages read 3270, created 13176, written 15109
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 8173, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
...
```

对于状态变量的描述，请参见第 7.1.10 节，“服务器状态变量”。

```sql
mysql> SHOW STATUS LIKE 'Innodb_buffer%';
+---------------------------------------+-------------+
| Variable_name                         | Value       |
+---------------------------------------+-------------+
| Innodb_buffer_pool_dump_status        | not started |
| Innodb_buffer_pool_load_status        | not started |
| Innodb_buffer_pool_resize_status      | not started |
| Innodb_buffer_pool_pages_data         | 8173        |
| Innodb_buffer_pool_bytes_data         | 133906432   |
| Innodb_buffer_pool_pages_dirty        | 0           |
| Innodb_buffer_pool_bytes_dirty        | 0           |
| Innodb_buffer_pool_pages_flushed      | 15109       |
| Innodb_buffer_pool_pages_free         | 1           |
| Innodb_buffer_pool_pages_misc         | 18          |
| Innodb_buffer_pool_pages_total        | 8192        |
| Innodb_buffer_pool_read_ahead_rnd     | 0           |
| Innodb_buffer_pool_read_ahead         | 2713        |
| Innodb_buffer_pool_read_ahead_evicted | 0           |
| Innodb_buffer_pool_read_requests      | 33069332    |
| Innodb_buffer_pool_reads              | 558         |
| Innodb_buffer_pool_wait_free          | 0           |
| Innodb_buffer_pool_write_requests     | 11985961    |
+---------------------------------------+-------------+
```
