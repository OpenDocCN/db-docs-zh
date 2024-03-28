# 28.4.3 INFORMATION_SCHEMA INNODB_BUFFER_PAGE_LRU 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-buffer-page-lru-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-buffer-page-lru-table.html)

`INNODB_BUFFER_PAGE_LRU` 表提供了关于 `InnoDB` 缓冲池 中页面的信息；特别是它们在 LRU 列表中的顺序，该列表确定了缓冲池在变满时要从中驱逐的页面。

`INNODB_BUFFER_PAGE_LRU` 表与 `INNODB_BUFFER_PAGE` 表具有相同的列，但有一些例外。它具有 `LRU_POSITION` 和 `COMPRESSED` 列，而不是 `BLOCK_ID` 和 `PAGE_STATE` 列，并且不包括 `IS_STALE` 列。

有关使用信息和示例，请参见第 17.15.5 节，“InnoDB INFORMATION_SCHEMA Buffer Pool Tables”。

警告

查询 `INNODB_BUFFER_PAGE_LRU` 表可能会影响性能。除非您了解性能影响并确定其可接受，否则不要在生产系统上查询此表。为避免在生产系统上影响性能，请在测试实例上重现您想要调查的问题，并查询缓冲池统计信息。

`INNODB_BUFFER_PAGE_LRU` 表具有以下列：

+   `POOL_ID`

    缓冲池 ID。这是一个标识符，用于区分多个缓冲池实例。

+   `LRU_POSITION`

    页在 LRU 列表中的位置。

+   `SPACE`

    表空间 ID；与 `INNODB_TABLES.SPACE` 相同的值。

+   `PAGE_NUMBER`

    页号。

+   `PAGE_TYPE`

    页类型。下表显示了允许的值。

    **表 28.6 INNODB_BUFFER_PAGE_LRU.PAGE_TYPE 值**

    | 页类型 | 描述 |
    | --- | --- |
    | `ALLOCATED` | 新分配的页面 |
    | `BLOB` | 未压缩的 BLOB 页面 |
    | `COMPRESSED_BLOB2` | 后续压缩 BLOB 页面 |
    | `COMPRESSED_BLOB` | 第一��压缩的 BLOB 页面 |
    | `ENCRYPTED_RTREE` | 加密的 R 树 |
    | `EXTENT_DESCRIPTOR` | 扩展描述符页面 |
    | `FILE_SPACE_HEADER` | 文件空间头 |
    | `FIL_PAGE_TYPE_UNUSED` | 未使用 |
    | `IBUF_BITMAP` | 插入缓冲位图 |
    | `IBUF_FREE_LIST` | 插入缓冲空闲列表 |
    | `IBUF_INDEX` | 插入缓冲索引 |
    | `INDEX` | B 树节点 |
    | `INODE` | 索引节点 |
    | `LOB_DATA` | 未压缩的 LOB 数据 |
    | `LOB_FIRST` | 未压缩 LOB 的第一页 |
    | `LOB_INDEX` | 未压缩 LOB 索引 |
    | `PAGE_IO_COMPRESSED` | 压缩页面 |
    | `PAGE_IO_COMPRESSED_ENCRYPTED` | 压缩且加密的页面 |
    | `PAGE_IO_ENCRYPTED` | 加密页面 |
    | `RSEG_ARRAY` | 回滚段数组 |
    | `RTREE_INDEX` | R 树索引 |
    | `SDI_BLOB` | 未压缩的 SDI BLOB |
    | `SDI_COMPRESSED_BLOB` | 压缩的 SDI BLOB |
    | `SDI_INDEX` | SDI 索引 |
    | `SYSTEM` | 系统页 |
    | `TRX_SYSTEM` | 事务系统数据 |
    | `UNDO_LOG` | 撤销日志页 |
    | `UNKNOWN` | 未知 |
    | `ZLOB_DATA` | 压缩 LOB 数据 |
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

    页面所属表的名称。此列仅适用于`PAGE_TYPE`值为`INDEX`的页面。如果服务器尚未访问表，则列为`NULL`。

+   `INDEX_NAME`

    页面所属索引的名称。这可以是聚簇索引或二级索引的名称。此列仅适用于`PAGE_TYPE`值为`INDEX`的页面。

+   `NUMBER_RECORDS`

    页面内记录的数量。

+   `DATA_SIZE`

    记录大小的总和。此列仅适用于`PAGE_TYPE`值为`INDEX`的页面。

+   `COMPRESSED_SIZE`

    压缩页面大小。对于未压缩的页面，为`NULL`。

+   `COMPRESSED`

    页面是否被压缩。

+   `IO_FIX`

    是否有任何 I/O 挂起在此页面：`IO_NONE` = 没有挂起的 I/O，`IO_READ` = 读取挂起，`IO_WRITE` = 写入挂起。

+   `IS_OLD`

    块是否在 LRU 列表中旧块的子列表中。

+   `FREE_PAGE_CLOCK`

    当块最后放置在 LRU 列表头部时，`freed_page_clock`计数器的值。`freed_page_clock`计数器跟踪从 LRU 列表末尾移除的块数。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE_LRU LIMIT 1\G
*************************** 1\. row ***************************
            POOL_ID: 0
       LRU_POSITION: 0
              SPACE: 97
        PAGE_NUMBER: 1984
          PAGE_TYPE: INDEX
         FLUSH_TYPE: 1
          FIX_COUNT: 0
          IS_HASHED: YES
NEWEST_MODIFICATION: 719490396
OLDEST_MODIFICATION: 0
        ACCESS_TIME: 3378383796
         TABLE_NAME: `employees`.`salaries`
         INDEX_NAME: PRIMARY
     NUMBER_RECORDS: 468
          DATA_SIZE: 14976
    COMPRESSED_SIZE: 0
         COMPRESSED: NO
             IO_FIX: IO_NONE
             IS_OLD: YES
    FREE_PAGE_CLOCK: 0
```

#### 注意

+   此表主要用于专家级性能监控，或者在为 MySQL 开发与性能相关的扩展时使用。

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。

+   查询此表可能需要 MySQL 分配一个大块连续内存，超过缓冲池中活动页面数量的 64 字节倍。这种分配可能会导致内存不足错误，特别是对于具有多千兆字节缓冲池的系统。

+   查询此表需要 MySQL 在遍历 LRU 列表时锁定表示缓冲池的数据结构，这可能会降低并发性，特别是对于具有多千兆字节缓冲池的系统。

+   当删除表、表行、分区或索引时，相关页面会保留在缓冲池中，直到为其他数据需要空间。`INNODB_BUFFER_PAGE_LRU`表报告有关这些页面的信息，直到它们从缓冲池中被驱逐。有关`InnoDB`如何管理缓冲池数据的更多信息，请参见 Section 17.5.1, “Buffer Pool”。
