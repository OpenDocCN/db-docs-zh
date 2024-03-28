# 28.4.4 INFORMATION_SCHEMA INNODB_BUFFER_POOL_STATS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-buffer-pool-stats-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-buffer-pool-stats-table.html)

`INNODB_BUFFER_POOL_STATS` 表提供了与 `SHOW ENGINE INNODB STATUS` 输出中提供的缓冲池信息相同的信息。也可以使用 `InnoDB` 缓冲池 服务器状态变量 获取相同的信息。

在缓冲池中使页面“年轻”或“非年轻”的概念是指在缓冲池数据结构的头部和尾部之间转移它们的过程。标记为“年轻”的页面需要更长时间才会从缓冲池中移除，而标记为“非年轻”的页面则更接近被 驱逐 的位置。

有关使用信息和示例，请参见 Section 17.15.5, “InnoDB INFORMATION_SCHEMA Buffer Pool Tables”。

[`INNODB_BUFFER_POOL_STATS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-buffer-pool-stats-table.html) 表包含以下列：

+   `POOL_ID`

    缓冲池 ID。这是用于区分多个缓冲池实例的标识符。

+   `POOL_SIZE`

    `InnoDB` 缓冲池大小（以页面为单位）。

+   `FREE_BUFFERS`

    `InnoDB` 缓冲池中的空闲页面数。

+   `DATABASE_PAGES`

    `InnoDB` 缓冲池中包含数据的页面数。这个数字包括脏页和干净页。

+   `OLD_DATABASE_PAGES`

    `old` 缓冲池子列表中的页面数。

+   `MODIFIED_DATABASE_PAGES`

    修改（脏）数据库页面数。

+   `PENDING_DECOMPRESS`

    等待解压的页面数。

+   `PENDING_READS`

    等待读取的页面数。

+   `PENDING_FLUSH_LRU`

    LRU 中等待刷新的页面数。

+   `PENDING_FLUSH_LIST`

    刷新列表中等待刷新的页面数。

+   `PAGES_MADE_YOUNG`

    标记为年轻的页面数。

+   `PAGES_NOT_MADE_YOUNG`

    未被标记为年轻的页面数。

+   `PAGES_MADE_YOUNG_RATE`

    每秒标记为年轻的页面数（自上次打印以来标记为年轻的页面数 / 经过的时间）。

+   `PAGES_MADE_NOT_YOUNG_RATE`

    每秒未被标记为年轻的页面数（自上次打印以来未被标记为年轻的页面数 / 经过的时间）。

+   `NUMBER_PAGES_READ`

    读取的页面数。

+   `NUMBER_PAGES_CREATED`

    创建的页面数。

+   `NUMBER_PAGES_WRITTEN`

    写入的页面数。

+   `PAGES_READ_RATE`

    每秒读取的页面数（自上次打印以来读取的页面数 / 经过的时间）。

+   `PAGES_CREATE_RATE`

    每秒创建的页面数量（自上次打印以来创建的页面/经过的时间）。

+   `PAGES_WRITTEN_RATE`

    每秒写入的页面数量（自上次打印以来写入的页面/经过的时间）。

+   `NUMBER_PAGES_GET`

    逻辑读请求的数量。

+   `HIT_RATE`

    缓冲池命中率。

+   `YOUNG_MAKE_PER_THOUSAND_GETS`

    每千次获取的页面变为年轻页面的数量。

+   `NOT_YOUNG_MAKE_PER_THOUSAND_GETS`

    每千次获取未变为年轻页面的数量。

+   `NUMBER_PAGES_READ_AHEAD`

    预读取的页面数量。

+   `NUMBER_READ_AHEAD_EVICTED`

    由预读取后台线程读入`InnoDB`缓冲池的页面数量，随后在没有被查询访问的情况下被驱逐。

+   `READ_AHEAD_RATE`

    每秒的预读取速率（自上次打印以来的预读取页面/经过的时间）。

+   `READ_AHEAD_EVICTED_RATE`

    每秒未访问的预读取页面被驱逐的数量（自上次打印以来未访问的预读取页面/经过的时间）。

+   `LRU_IO_TOTAL`

    总 LRU I/O。

+   `LRU_IO_CURRENT`

    当前间隔的 LRU I/O。

+   `UNCOMPRESS_TOTAL`

    解压缩的页面总数。

+   `UNCOMPRESS_CURRENT`

    当前间隔内解压缩的页面数量。

#### 示例。

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_BUFFER_POOL_STATS\G
*************************** 1\. row ***************************
                         POOL_ID: 0
                       POOL_SIZE: 8192
                    FREE_BUFFERS: 1
                  DATABASE_PAGES: 8085
              OLD_DATABASE_PAGES: 2964
         MODIFIED_DATABASE_PAGES: 0
              PENDING_DECOMPRESS: 0
                   PENDING_READS: 0
               PENDING_FLUSH_LRU: 0
              PENDING_FLUSH_LIST: 0
                PAGES_MADE_YOUNG: 22821
            PAGES_NOT_MADE_YOUNG: 3544303
           PAGES_MADE_YOUNG_RATE: 357.62602199870594
       PAGES_MADE_NOT_YOUNG_RATE: 0
               NUMBER_PAGES_READ: 2389
            NUMBER_PAGES_CREATED: 12385
            NUMBER_PAGES_WRITTEN: 13111
                 PAGES_READ_RATE: 0
               PAGES_CREATE_RATE: 0
              PAGES_WRITTEN_RATE: 0
                NUMBER_PAGES_GET: 33322210
                        HIT_RATE: 1000
    YOUNG_MAKE_PER_THOUSAND_GETS: 18
NOT_YOUNG_MAKE_PER_THOUSAND_GETS: 0
         NUMBER_PAGES_READ_AHEAD: 2024
       NUMBER_READ_AHEAD_EVICTED: 0
                 READ_AHEAD_RATE: 0
         READ_AHEAD_EVICTED_RATE: 0
                    LRU_IO_TOTAL: 0
                  LRU_IO_CURRENT: 0
                UNCOMPRESS_TOTAL: 0
              UNCOMPRESS_CURRENT: 0
```

#### 注意事项。

+   此表主要用于专家级性能监控，或者在开发与 MySQL 性能相关的扩展时使用。

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。
