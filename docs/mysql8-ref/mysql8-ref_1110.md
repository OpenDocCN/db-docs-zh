> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-slave-hosts.html`](https://dev.mysql.com/doc/refman/8.0/en/show-slave-hosts.html)

#### 15.7.7.34 展示从机主机 | 展示副本语句

```sql
{SHOW SLAVE HOSTS | SHOW REPLICAS}
```

显示当前在源端注册的副本列表。从 MySQL 8.0.22 开始，`展示从机主机`已被弃用，应改用别名 `展示副本`。该语句的工作方式与以前相同，只是语句及其输出所使用的术语已更改。在使用时，两个版本的语句都会更新相同的状态变量。请参阅 `展示副本` 的文档以获取语句的描述。
