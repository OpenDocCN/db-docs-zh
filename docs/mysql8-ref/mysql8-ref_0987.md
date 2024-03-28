# 15.4.1 用于控制源服务器的 SQL 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-statements-master.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-statements-master.html)

15.4.1.1 清除二进制日志语句

15.4.1.2 重置主服务器语句

15.4.1.3 设置 sql_log_bin 语句

本节讨论了用于管理复制源服务器的语句。第 15.4.2 节，“用于控制副本服务器的 SQL 语句”，讨论了用于管理副本服务器的语句。

除了这里描述的语句外，以下`SHOW`语句用于在复制中与源服务器一起使用。有关这些语句的信息，请参阅第 15.7.7 节，“显示语句”。

+   `SHOW BINARY LOGS`

+   `SHOW BINLOG EVENTS`

+   `SHOW MASTER STATUS`

+   `SHOW REPLICAS`（或在 MySQL 8.0.22 之前，`SHOW SLAVE HOSTS`）
