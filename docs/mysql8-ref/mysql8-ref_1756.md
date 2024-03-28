> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-blocks.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-blocks.html)

#### 25.6.16.5 ndbinfo 表

`blocks` 表是一个静态表，仅包含所有 NDB 内核块的名称和内部 ID（参见 NDB 内核块）。它供其他`ndbinfo`表（其中大部分实际上是视图）使用，以将块号映射到块名称，以生成人类可读的输出。

`blocks` 表包含以下列：

+   `block_number`

    块号

+   `block_name`

    块名称

##### 笔记

要获取所有块名称的列表，只需执行`SELECT block_name FROM ndbinfo.blocks`。尽管这是一个静态表，但其内容可能会因不同的 NDB Cluster 版本而有所不同。
