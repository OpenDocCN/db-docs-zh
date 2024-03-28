# 15.4.2 用于控制 REPLICA 服务器的 SQL 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-statements-replica.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-statements-replica.html)

15.4.2.1 更改主服务器 TO 语句

15.4.2.2 更改复制过滤器语句

15.4.2.3 更改复制源 TO 语句

15.4.2.4 重置 REPLICA 语句

15.4.2.5 重置 SLAVE 语句

15.4.2.6 启动 REPLICA 语句

15.4.2.7 启动 SLAVE 语句

15.4.2.8 停止 REPLICA 语句

15.4.2.9 停止 SLAVE 语句

本节讨论了用于管理 REPLICA 服务器的语句。第 15.4.1 节，“用于控制源服务器的 SQL 语句”，讨论了用于管理源服务器的语句。

除了这里描述的语句外，`SHOW REPLICA STATUS` 和 `SHOW RELAYLOG EVENTS` 也与 REPLICA 一起使用。有关这些语句的信息，请参阅 第 15.7.7.35 节，“显示 REPLICA 状态语句” 和 第 15.7.7.32 节，“显示 RELAYLOG 事件语句”。
