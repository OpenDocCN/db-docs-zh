# 28.4.26 INFORMATION_SCHEMA INNODB_TABLESTATS 视图

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-tablestats-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-tablestats-table.html)

`INNODB_TABLESTATS` 表提供了关于 `InnoDB` 表的低级状态信息的视图。这些数据由 MySQL 优化器使用，以计算在查询 `InnoDB` 表时要使用哪个索引。这些信息源自内存数据结构，而不是存储在磁盘上的数据。没有相应的内部 `InnoDB` 系统表。

如果自上次服务器重启以来已打开并且尚未从表缓存中过期的话，`InnoDB` 表将在此视图中表示。始终在此视图中表示具有持久统计信息的表。

仅对修改索引列的 `DELETE` 或 `UPDATE` 操作更新表统计信息。仅修改非索引列的操作不会更新统计信息。

`ANALYZE TABLE` 清除表统计信息，并将 `STATS_INITIALIZED` 列设置为 `Uninitialized`。下次访问表时将重新收集统计信息。

有关相关用法信息和示例，请参见 第 17.15.3 节，“InnoDB INFORMATION_SCHEMA Schema Object Tables”。

`INNODB_TABLESTATS` 表具有以下列：

+   `TABLE_ID`

    表的标识符，可用于查看可用统计信息的表；与 `INNODB_TABLES.TABLE_ID` 相同的值。

+   `NAME`

    表的名称；与 `INNODB_TABLES.NAME` 相同的值。

+   `STATS_INITIALIZED`

    如果已经收集了统计信息，则值为 `Initialized`，否则为 `Uninitialized`。

+   `NUM_ROWS`

    表中当前估计的行数。每次 DML 操作后更新。如果未提交事务正在向表中插入或删除数据，则该值可���不准确。

+   `CLUST_INDEX_SIZE`

    存储聚簇索引的磁盘上的页数，该索引按主键顺序保存 `InnoDB` 表数据。如果尚未为表收集统计信息，则此值可能为 null。

+   `OTHER_INDEX_SIZE`

    存储表的所有辅助索引的磁盘上的页数。如果尚未为表收集统计信息，则此值可能为 null。

+   `MODIFIED_COUNTER`

    被 DML 操作修改的行数，如 `INSERT`、`UPDATE`、`DELETE`，还有外键级联操作。每次重新计算表统计信息时，此列将被重置。

+   `AUTOINC`

    任何基于自增操作的下一个要发行的数字。`AUTOINC` 值变化的速率取决于自增数已被请求多少次以及每次请求被授予多少个数字。

+   `REF_COUNT`

    当这个计数器达到零时，表元数据可以从表缓存中驱逐出去。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLESTATS where TABLE_ID = 71\G
*************************** 1\. row ***************************
         TABLE_ID: 71
             NAME: test/t1
STATS_INITIALIZED: Initialized
         NUM_ROWS: 1
 CLUST_INDEX_SIZE: 1
 OTHER_INDEX_SIZE: 0
 MODIFIED_COUNTER: 1
          AUTOINC: 0
        REF_COUNT: 1
```

#### 注意

+   这个表主要用于专家级性能监控，或者在开发与 MySQL 相关的性能扩展时使用。

+   你必须拥有`PROCESS`权限才能查询这个表。

+   使用 `INFORMATION_SCHEMA` `COLUMNS` 表或 `SHOW COLUMNS` 语句查看关于这个表的列的额外信息，包括数据类型和默认值。
