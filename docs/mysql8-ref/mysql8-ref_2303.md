> 原文：[`dev.mysql.com/doc/refman/8.0/en/out-of-memory.html`](https://dev.mysql.com/doc/refman/8.0/en/out-of-memory.html)

#### B.3.2.6 内存不足

如果你使用**mysql**客户端程序发出查询，并收到以下错误，意味着**mysql**没有足够的内存来存储整个查询结果：

```sql
mysql: Out of memory at line 42, 'malloc.c'
mysql: needed 8136 byte (8k), memory in use: 12481367 bytes (12189k)
ERROR 2008: MySQL client ran out of memory
```

要解决这个问题，首先检查一下你的查询是否正确。返回这么多行是否合理？如果不合理，修改查询然后再试一次。否则，你可以使用带有`--quick`选项的**mysql**。这会导致它使用`mysql_use_result()` C API 函数来检索结果集，这样会减轻客户端的负担（但增加服务器的负担）。
