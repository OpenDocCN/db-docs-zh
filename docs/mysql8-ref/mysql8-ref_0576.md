# 10.4.5 数据库和表数量的限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/database-count-limit.html`](https://dev.mysql.com/doc/refman/8.0/en/database-count-limit.html)

MySQL 对数据库的数量没有限制。底层文件系统可能对目录数量有限制。

MySQL 对表的数量没有限制。底层文件系统可能对代表表的文件数量有限制。各个存储引擎可能会施加特定于引擎的限制。`InnoDB`允许最多 4 十亿个表。
