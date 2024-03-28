# 29.12.13 性能模式锁表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-lock-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-lock-tables.html)

29.12.13.1 数据锁表

29.12.13.2 数据锁等待表

29.12.13.3 元数据锁表

29.12.13.4 表句柄表

性能模式通过这些表公开锁信息：

+   `data_locks`: 持有和请求的数据锁

+   `data_lock_waits`: 数据锁所有者和被这些所有者阻塞的数据锁请求者之间的关系

+   `metadata_locks`: 持有和请求的元数据锁

+   `table_handles`: 持有和请求的表锁

以下部分详细描述这些表。
