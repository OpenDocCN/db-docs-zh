# 14.20.5 窗口函数限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/window-function-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/window-function-restrictions.html)

SQL 标准对窗口函数施加了一个限制，即它们不能在 `UPDATE` 或 `DELETE` 语句中用于更新行。在这些语句的子查询中使用这些函数（选择行）是允许的。

MySQL 不支持这些窗口函数特性：

+   `DISTINCT` 语法用于聚合窗口函数。

+   嵌套窗口函数。

+   依赖于当前行值的动态帧端点。

解析器识别这些窗口构造，但仍不支持：

+   `GROUPS` 帧单位说明符被解析，但会产生错误。只支持 `ROWS` 和 `RANGE`。

+   解析帧规范的 `EXCLUDE` 子句，但会产生错误。

+   `IGNORE NULLS` 被解析，但会产生错误。只支持 `RESPECT NULLS`。

+   `FROM LAST` 被解析，但会产生错误。只支持 `FROM FIRST`。

截至 MySQL 8.0.28，对于给定的 `SELECT`，支持最多 127 个窗口。请注意，单个查询可能使用多个 `SELECT` 子句，每个子句支持最多 127 个窗口。不同窗口的数量定义为命名窗口的总和以及任何作为任何窗口函数的 `OVER` 子句的一部分指定的隐式窗口。您还应该注意，使用大量窗口的查询可能需要增加默认线程堆栈大小（`thread_stack` 系统变量）。
