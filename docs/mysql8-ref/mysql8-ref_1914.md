# 28.3.19 INFORMATION_SCHEMA OPTIMIZER_TRACE 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-optimizer-trace-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-optimizer-trace-table.html)

`OPTIMIZER_TRACE` 表提供了由优化器追踪功能为被追踪语句生成的信息。要启用跟踪，请使用`optimizer_trace` 系统变量。有关详细信息，请参阅 MySQL 内部：跟踪优化器。

`OPTIMIZER_TRACE` 表包含以下列：

+   `QUERY`

    被追踪语句的文本。

+   `TRACE`

    追踪内容，以`JSON`格式。

+   `MISSING_BYTES_BEYOND_MAX_MEM_SIZE`

    每个记忆的追踪都是一个字符串，随着优化的进行而扩展并向其附加数据。`optimizer_trace_max_mem_size` 变量设置了当前所有记忆追踪使用的内存总量的限制。如果达到此限制，当前追踪不会被扩展（因此是不完整的），`MISSING_BYTES_BEYOND_MAX_MEM_SIZE`列显示了追踪中缺少的字节数。

+   `INSUFFICIENT_PRIVILEGES`

    如果被追踪的查询使用了具有`SQL SECURITY`值为`DEFINER`的视图或存储过程，可能会导致除定义者以外的用户无法查看查询的追踪。在这种情况下，追踪显示为空，`INSUFFICIENT_PRIVILEGES`的值为 1。否则，该值为 0。
