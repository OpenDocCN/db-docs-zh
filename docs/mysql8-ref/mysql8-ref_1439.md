> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-drop-if-exists.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-drop-if-exists.html)

#### 19.5.1.11 复制`DROP ... IF EXISTS`语句

`DROP DATABASE IF EXISTS`，`DROP TABLE IF EXISTS`和`DROP VIEW IF EXISTS`语句始终会被复制，即使要删除的数据库、表或视图在源上不存在。这是为了确保一旦副本赶上源，要删除的对象在源或副本上都不再存在。

`DROP ... IF EXISTS` 语句用于存储程序（存储过程和函数，触发器和事件），即使要删除的存储程序在源上不存在，也会被复制。
