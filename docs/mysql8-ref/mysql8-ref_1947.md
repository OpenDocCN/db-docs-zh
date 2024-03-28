# 28.4.1 INFORMATION_SCHEMA InnoDB 表参考

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-table-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-table-reference.html)

以下表格总结了 `INFORMATION_SCHEMA` 中的 InnoDB 表。更详细的信息，请参阅各个表的描述。

**表 28.3 INFORMATION_SCHEMA InnoDB 表**

| 表名 | 描述 | 引入版本 |
| --- | --- | --- |
| `INNODB_BUFFER_PAGE` | InnoDB 缓冲池中的页面 |  |
| `INNODB_BUFFER_PAGE_LRU` | InnoDB 缓冲池中页面的 LRU 排序 |  |
| `INNODB_BUFFER_POOL_STATS` | InnoDB 缓冲池统计信息 |  |
| `INNODB_CACHED_INDEXES` | InnoDB 缓冲池中每个索引缓存的索引页数 |  |
| `INNODB_CMP` | 与压缩的 InnoDB 表相关的操作状态 |  |
| `INNODB_CMP_PER_INDEX` | 与压缩的 InnoDB 表和索引相关的操作状态 |  |
| `INNODB_CMP_PER_INDEX_RESET` | 与压缩的 InnoDB 表和索引相关的操作状态 |  |
| `INNODB_CMP_RESET` | 与压缩的 InnoDB 表相关的操作状态 |  |
| `INNODB_CMPMEM` | InnoDB 缓冲池内压缩页面的状态 |  |
| `INNODB_CMPMEM_RESET` | InnoDB 缓冲池内压缩页面的状态 |  |
| `INNODB_COLUMNS` | 每个 InnoDB 表中的列 |  |
| `INNODB_DATAFILES` | InnoDB 文件表和通用表空间的数据文件路径信息 |  |
| `INNODB_FIELDS` | InnoDB 索引的关键列 |  |
| `INNODB_FOREIGN` | InnoDB 外键元数据 |  |
| `INNODB_FOREIGN_COLS` | InnoDB 外键列状态信息 |  |
| `INNODB_FT_BEING_DELETED` | INNODB_FT_DELETED 表的快照 |  |
| `INNODB_FT_CONFIG` | InnoDB 表全文索引和相关处理的元数据 |  |
| `INNODB_FT_DEFAULT_STOPWORD` | InnoDB 全文索引的默认停用词列表 |  |
| `INNODB_FT_DELETED` | 从 InnoDB 表全文索引中删除的行 |  |
| `INNODB_FT_INDEX_CACHE` | InnoDB 全文索引中新插入行的标记信息 |  |
| `INNODB_FT_INDEX_TABLE` | 用于处理针对 InnoDB 表全文索引的文本搜索的倒排索引信息 |  |
| `INNODB_INDEXES` | InnoDB 索引元数据 |  |
| `INNODB_METRICS` | InnoDB 性能信息 |  |
| `INNODB_SESSION_TEMP_TABLESPACES` | 会话临时表空间元数据 | 8.0.13 |
| `INNODB_TABLES` | InnoDB 表元数据 |  |
| `INNODB_TABLESPACES` | InnoDB 按表存储、通用和撤销表空间元数据 |  |
| `INNODB_TABLESPACES_BRIEF` | 简要��按表存储、通用、撤销和系统表空间元数据 |  |
| `INNODB_TABLESTATS` | InnoDB 表低级状态信息 |  |
| `INNODB_TEMP_TABLE_INFO` | 关于活跃的用户创建的 InnoDB 临时表的信息 |  |
| `INNODB_TRX` | 活跃的 InnoDB 事务信息 |  |
| `INNODB_VIRTUAL` | InnoDB 虚拟生成列元数据 |  |
| 表名 | 描述 | 引入版本 |
