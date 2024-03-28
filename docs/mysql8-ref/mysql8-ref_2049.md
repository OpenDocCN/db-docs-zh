# 29.12.10 性能模式用户定义变量表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-user-variable-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-user-variable-tables.html)

性能模式提供了一个`user_variables_by_thread`表，用于公开用户定义的变量。这些变量是在特定会话中定义的，并且在名称前面带有`@`字符；参见第 11.4 节，“用户定义变量”。

`user_variables_by_thread`表具有以下列：

+   `THREAD_ID`

    会话中定义变量的线程标识符。

+   `VARIABLE_NAME`

    变量名称，不包括前导的`@`字符。

+   `VARIABLE_VALUE`

    变量值。

`user_variables_by_thread`表具有以下索引：

+   主键为(`THREAD_ID`, `VARIABLE_NAME`)

`TRUNCATE TABLE`不允许用于`user_variables_by_thread`表。
