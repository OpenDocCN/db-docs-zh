# 10.8 理解查询执行计划

> 原文：[`dev.mysql.com/doc/refman/8.0/en/execution-plan-information.html`](https://dev.mysql.com/doc/refman/8.0/en/execution-plan-information.html)

10.8.1 使用 EXPLAIN 优化查询

10.8.2 EXPLAIN 输出格式

10.8.3 扩展的 EXPLAIN 输出格式

10.8.4 获取命名连接的执行计划信息

10.8.5 估算查询性能

根据您的表、列、索引的细节以及`WHERE`子句中的条件，MySQL 优化器考虑许多技术来高效执行 SQL 查询中涉及的查找操作。对于大表的查询可以在不读取所有行的情况下执行；涉及多个表的连接可以在不比较每个行的组合的情况下执行。优化器选择执行最有效查询的一组操作称为“查询执行计划”，也称为`EXPLAIN`计划。您的目标是识别`EXPLAIN`计划中表明查询已经优化良好的方面，并学习 SQL 语法和索引技术，以改进计划，如果您看到一些低效的操作。
