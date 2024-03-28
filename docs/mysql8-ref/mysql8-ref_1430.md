> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-blackhole.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-blackhole.html)

#### 19.5.1.2 复制和 BLACKHOLE 表

`BLACKHOLE`存储引擎接受数据但会丢弃它，不会存储。在执行二进制日志记录时，所有对这些表的插入操作都会被记录，无论使用的日志格式是什么。根据使用的基于语句或基于行的日志记录方式，更新和删除操作会有不同的处理方式。在基于语句的日志记录格式下，所有影响`BLACKHOLE`表的语句都会被记录，但它们的效果会被忽略。在使用基于行的日志记录时，对这些表的更新和删除操作会被简单地跳过，不会被写入二进制日志。每当发生这种情况时，都会记录一个警告。

出于这个原因，我们建议当您使用`BLACKHOLE`存储引擎复制表时，将`binlog_format`服务器变量设置为`STATEMENT`，而不是`ROW`或`MIXED`。
