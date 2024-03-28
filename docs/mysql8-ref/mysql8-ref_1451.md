> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-optimizer.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-optimizer.html)

#### 19.5.1.23 复制和查询优化器

如果一个语句被编写成非确定性的方式，即由查询优化器决定，那么源数据库和副本数据库上的数据可能会变得不同。（一般来说，这不是一个好的做法，即使在复制之外也是如此。）非确定性语句的例子包括使用`LIMIT`而没有`ORDER BY`子句的`DELETE`或`UPDATE`语句；详细讨论请参见 Section 19.5.1.18, “Replication and LIMIT”。
