# 28.4.2 INFORMATION_SCHEMA INNODB_BUFFER_PAGE 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-buffer-page-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-buffer-page-table.html)

`INNODB_BUFFER_PAGE`表提供了关于`InnoDB`缓冲池中每个页面的信息。

有关相关用法信息和示例，请参见第 17.15.5 节，“InnoDB INFORMATION_SCHEMA 缓冲池表”。

警告

查询`INNODB_BUFFER_PAGE`表可能会影响性能。除非您了解性能影响并确定其可接受，否则不要在生产系统上查询此表。为避免影响生产系统性能，请在测试实例上重现要调查的问题并查询缓冲池统计信息。

`INNODB_BUFFER_PAGE`表具有以下列：

+   `POOL_ID`

    缓冲池 ID。这是一个标识符，用于区分多个缓冲池实例。

+   `块 ID`

    缓冲池块 ID。

+   `空间`

    表空间 ID；与`INNODB_TABLES.SPACE`相同。

+   `PAGE_NUMBER`

    页码。

+   `PAGE_TYPE`

    页面类型。以下表显示了允许的值。

    **表 28.4 INNODB_BUFFER_PAGE.PAGE_TYPE 值**

    | 页面类型 | 描述 |
    | --- | --- |
    | `ALLOCATED` | 新分配的页面 |
    | `BLOB` | 未压缩的 BLOB 页面 |
    | `COMPRESSED_BLOB2` | 后续压缩 BLOB 页面 |
    | `COMPRESSED_BLOB` | 第一个压缩的 BLOB 页面 |
    | `ENCRYPTED_RTREE` | 加密的 R 树 |
    | `EXTENT_DESCRIPTOR` | 扩展描述符页面 |
    | `FILE_SPACE_HEADER` | 文件空间头 |
    | `FIL_PAGE_TYPE_UNUSED` | 未使用 |
    | `IBUF_BITMAP` | 插入缓冲位图 |
    | `IBUF_FREE_LIST` | 插入缓冲区空闲列表 |
    | `IBUF_INDEX` | 插入缓冲索引 |
    | `INDEX` | B 树节点 |
    | `INODE` | 索引节点 |
    | `LOB_DATA` | 未压缩的 LOB 数据 |
    | `LOB_FIRST` | 未压缩 LOB 的第一页 |
    | `LOB_INDEX` | 未压缩的 LOB 索引 |
    | `PAGE_IO_COMPRESSED` | 压缩页面 |
    | `PAGE_IO_COMPRESSED_ENCRYPTED` | 压缩和加密页面 |
    | `PAGE_IO_ENCRYPTED` | 加密页面 |
    | `RSEG_ARRAY` | 回滚段数组 |
    | `RTREE_INDEX` | R 树索引 |
    | `SDI_BLOB` | 未压缩的 SDI BLOB |
    | `SDI_COMPRESSED_BLOB` | 压缩的 SDI BLOB |
    | `SDI_INDEX` | SDI 索引 |
    | `SYSTEM` | 系统页面 |
    | `TRX_SYSTEM` | 事务系统数据 |
    | `UNDO_LOG` | 撤销日志页面 |
    | `UNKNOWN` | 未知 |
    | `ZLOB_DATA` | 压缩的 LOB 数据 |
    | `ZLOB_FIRST` | 压缩 LOB 的第一页 |
    | `ZLOB_FRAG` | 压缩 LOB 片段 |
    | `ZLOB_FRAG_ENTRY` | 压缩 LOB 片段索引 |
    | `ZLOB_INDEX` | 压缩 LOB 索引 |
    | 页面类型 | 描述 |

+   `FLUSH_TYPE`

    刷新类型。

+   `FIX_COUNT`

    在缓冲池中使用此块的线程数。当为零时，该块有资格被驱逐。

+   `IS_HASHED`

    是否在此页面上构建了哈希索引。

+   `NEWEST_MODIFICATION`

    最新修改的日志序列号。

+   `OLDEST_MODIFICATION`

    最旧修改的日志序列号。

+   `ACCESS_TIME`

    用于判断页面首次访问时间的抽象数字。

+   `TABLE_NAME`

    页面所属的表的名称。此列仅适用于`PAGE_TYPE`值为`INDEX`的页面。如果服务器尚未访问表，则该列为`NULL`。

+   `INDEX_NAME`

    页面所属的索引的名称。这可以是聚簇索引或二级索引的名称。此列仅适用于`PAGE_TYPE`值为`INDEX`的页面。

+   `NUMBER_RECORDS`

    页面内记录的数量。

+   `DATA_SIZE`

    记录大小的总和。此列仅适用于`PAGE_TYPE`值为`INDEX`的页面。

+   `COMPRESSED_SIZE`

    压缩页面大小。对于未压缩的页面，为`NULL`。

+   `PAGE_STATE`

    页面状态。下表显示了允许的值。

    **表 28.5 INNODB_BUFFER_PAGE.PAGE_STATE 值**

    | 页面状态 | 描述 |
    | --- | --- |
    | `FILE_PAGE` | 缓冲文件页面 |
    | `MEMORY` | 包含主内存对象 |
    | `NOT_USED` | 在空闲列表中 |
    | `NULL` | 清洁的压缩页面，刷新列表中的压缩页面，用作缓冲池监视哨的页面 |
    | `READY_FOR_USE` | 空闲页面 |
    | `REMOVE_HASH` | 在放入空闲列表之前应删除哈希索引 |

+   `IO_FIX`

    此页面是否有任何 I/O 挂起：`IO_NONE` = 没有挂起的 I/O，`IO_READ` = 读挂起，`IO_WRITE` = 写挂起，`IO_PIN` = 禁止重新定位和从刷新中移除。

+   `IS_OLD`

    块是否在 LRU 列表中旧块的子列表中。

+   `FREE_PAGE_CLOCK`

    当块最后放置在 LRU 列表头部时，`freed_page_clock`计数器的值。`freed_page_clock`计数器跟踪从 LRU 列表末尾移除的块数。

+   `IS_STALE`

    页面是否过时。在 MySQL 8.0.24 中添加。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE LIMIT 1\G
*************************** 1\. row ***************************
            POOL_ID: 0
           BLOCK_ID: 0
              SPACE: 97
        PAGE_NUMBER: 2473
          PAGE_TYPE: INDEX
         FLUSH_TYPE: 1
          FIX_COUNT: 0
          IS_HASHED: YES
NEWEST_MODIFICATION: 733855581
OLDEST_MODIFICATION: 0
        ACCESS_TIME: 3378385672
         TABLE_NAME: `employees`.`salaries`
         INDEX_NAME: PRIMARY
     NUMBER_RECORDS: 468
          DATA_SIZE: 14976
    COMPRESSED_SIZE: 0
         PAGE_STATE: FILE_PAGE
             IO_FIX: IO_NONE
             IS_OLD: YES
    FREE_PAGE_CLOCK: 66
           IS_STALE: NO
```

#### 注意

+   此表主要用于专家级性能监控，或者在为 MySQL 开发与性能相关的扩展时使用。

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表列的其他信息，包括数据类型和默认值。

+   当删除表、表行、分区或索引时，相关页面会保留在缓冲池中，直到需要空间存储其他数据。`INNODB_BUFFER_PAGE` 表报告这些页面的信息，直到它们从缓冲池中被驱逐。有关`InnoDB`如何管理缓冲池数据的更多信息，请参见第 17.5.1 节，“缓冲池”。
