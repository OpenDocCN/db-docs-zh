# 10.4.3 优化多表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimize-multi-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/optimize-multi-tables.html)

10.4.3.1 MySQL 如何打开和关闭表

10.4.3.2 在同一数据库中创建许多表的缺点

保持单个查询快速的一些技术涉及将数据分布在许多表中。当表的数量达到数千甚至数百万时，处理所有这些表的开销成为一个新的性能考虑因素。
