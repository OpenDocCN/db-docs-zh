> 原文：[`dev.mysql.com/doc/refman/8.0/en/cursor-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/cursor-restrictions.html)

#### 15.6.6.5 服务器端游标的限制

服务器端游标在 C API 中使用`mysql_stmt_attr_set()`函数实现。存储过程中的游标也使用相同的实现。服务器端游标使得可以在服务器端生成结果集，但除了客户端请求的行之外，不会将其传输到客户端。例如，如果客户端执行查询但只对第一行感兴趣，则不会传输其余行。

在 MySQL 中，服务器端游标被实例化为内部临时表。最初，这是一个`MEMORY`表，但当其大小超过`max_heap_table_size`和`tmp_table_size`系统变量的最小值时，将转换为`MyISAM`表。为了保存游标的结果集而创建的内部临时表与其他用途的内部临时表一样受到相同的限制。参见 Section 10.4.4, “Internal Temporary Table Use in MySQL”。实现的一个限制是，对于大型结果集，通过游标检索其行可能会很慢。

游标是只读的；不能使用游标更新行。

`UPDATE WHERE CURRENT OF`和`DELETE WHERE CURRENT OF`未实现，因为不支持可更新游标。

游标不可保持（在提交后不保持打开状态）。

游标是不敏感的。

游标是不可滚动的。

游标没有名称。语句处理程序充当游标 ID。

每个准备语句只能打开一个游标。如果需要多个游标，必须准备多个语句。

如果不支持准备模式中的语句，则不能对生成结果集的语句使用游标。这包括`CHECK TABLE`、`HANDLER READ`和`SHOW BINLOG EVENTS`等语句。
