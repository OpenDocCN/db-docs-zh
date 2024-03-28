> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-partitioning.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-partitioning.html)

#### 19.5.1.24 复制和分区

只要分区表使用相同的分区方案并且结构相同，复制就支持分区表之间的复制，除非特别允许异常情况（参见 Section 19.5.1.9, “源表和副本表上定义不同的复制”）。

不同分区的表之间的复制通常不受支持。这是因为在这种情况下直接作用于分区的语句（比如`ALTER TABLE ... DROP PARTITION`）可能会在源表和副本表上产生不同的结果。在源表分区但副本表未分区的情况下，任何在源表副本上操作分区的语句都会在副本表上失败。当副本表分区但源表未分区时，在源表上运行直接作用于分区的语句会导致错误。为避免停止复制或在源表和副本表之间创建不一致，始终确保源表和副本表的对应复制表以相同方式分区。
