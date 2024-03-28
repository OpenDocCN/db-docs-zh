# 10.2.5 优化数据更改语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-change-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/data-change-optimization.html)

10.2.5.1 优化 INSERT 语句

10.2.5.2 优化 UPDATE 语句

10.2.5.3 优化 DELETE 语句

本节解释了如何加快数据更改语句：`INSERT`，`UPDATE`和`DELETE`。传统的 OLTP 应用程序和现代的 Web 应用程序通常执行许多小型数据更改操作，其中并发性至关重要。数据分析和报告应用程序通常运行影响许多行的数据更改操作，其中主要考虑因素是写入大量数据的 I/O 和保持索引最新。对于插入和更新大量数据（在行业中称为 ETL，即“提取-转换-加载”），有时您会使用其他 SQL 语句或外部命令，模仿`INSERT`，`UPDATE`和`DELETE`语句的效果。
