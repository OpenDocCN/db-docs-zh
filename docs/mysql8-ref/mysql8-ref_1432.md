> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-checksum-table.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-checksum-table.html)

#### 19.5.1.4 复制和 CHECKSUM TABLE

`CHECKSUM TABLE` 返回一个逐行计算的校验和，使用依赖于表行存储格式的方法。存储格式在 MySQL 版本之间不保证保持不变，因此在升级后校验和值可能会改变。
