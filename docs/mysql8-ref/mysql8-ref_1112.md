> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-slave-status.html`](https://dev.mysql.com/doc/refman/8.0/en/show-slave-status.html)

#### 15.7.7.36 展示从属 | 复制状态语句

```sql
SHOW {SLAVE | REPLICA} STATUS [FOR CHANNEL *channel*]
```

该语句提供了关于从属线程的关键参数状态信息。从 MySQL 8.0.22 开始，`SHOW SLAVE STATUS` 已被弃用，应改用别名 `SHOW REPLICA STATUS`。该语句的工作方式与以前相同，只是语句及其输出所使用的术语已更改。使用两个版本的语句时，它们会更新相同的状态变量。请参阅 `SHOW REPLICA STATUS` 的文档以获取语句的描述。
