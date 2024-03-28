# 10.10.1 InnoDB 缓冲池优化

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool-optimization.html)

`InnoDB`维护一个称为缓冲池的存储区域，用于在内存中缓存数据和索引。了解`InnoDB`缓冲池的工作原理，并利用它将频繁访问的数据保留在内存中，是 MySQL 调优的重要方面。

有关`InnoDB`缓冲池内部工作原理、LRU 替换算法概述和一般配置信息的解释，请参阅第 17.5.1 节，“缓冲池”。

有关额外的`InnoDB`缓冲池配置和调优信息，请参阅以下章节：

+   第 17.8.3.4 节，“配置 InnoDB 缓冲池预取（预读）”

+   第 17.8.3.5 节，“配置缓冲池刷新”

+   第 17.8.3.3 节，“使缓冲池具有扫描抵抗性”

+   第 17.8.3.2 节，“配置多个缓冲池实例”

+   第 17.8.3.6 节，“保存和恢复缓冲池状态”

+   第 17.8.3.1 节，“配置 InnoDB 缓冲池大小”
