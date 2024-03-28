> 原文：[`dev.mysql.com/doc/refman/8.0/en/set-character-set.html`](https://dev.mysql.com/doc/refman/8.0/en/set-character-set.html)

#### 15.7.6.2 SET CHARACTER SET Statement

```sql
SET {CHARACTER SET | CHARSET}
    {'*charset_name*' | DEFAULT}
```

此语句将服务器和当前客户端之间发送的所有字符串与给定映射进行映射。`SET CHARACTER SET`设置三个会话系统变量：`character_set_client`和`character_set_results`设置为给定的字符集，`character_set_connection`设置为`character_set_database`的值。请参阅第 12.4 节，“连接字符集和校对”。

*`charset_name`*可以带引号或不带引号。

默认字符集映射可以通过使用值`DEFAULT`来恢复。默认值取决于服务器配置。

一些字符集不能作为客户端字符集使用。尝试与`SET CHARACTER SET`一起使用会产生错误。请参阅不允许的客户端字符集。
