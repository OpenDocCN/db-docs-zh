> 原文：[`dev.mysql.com/doc/refman/8.0/en/reset-slave.html`](https://dev.mysql.com/doc/refman/8.0/en/reset-slave.html)

#### 15.4.2.5 重置从库语句

```sql
RESET {SLAVE | REPLICA} [ALL] [*channel_option*]

*channel_option*:
    FOR CHANNEL *channel*
```

使复制品忘记其在源二进制日志中的位置。从 MySQL 8.0.22 开始，`RESET SLAVE`已被弃用，应改用别名`RESET REPLICA`。在 MySQL 8.0.22 之前的版本中，请使用`RESET SLAVE`。该语句的工作方式与以前相同，只是语句及其输出所使用的术语已更改。使用时，两个版本的语句会更新相同的状态变量。请参阅`RESET REPLICA`的文档以获取有关该语句的描述。
