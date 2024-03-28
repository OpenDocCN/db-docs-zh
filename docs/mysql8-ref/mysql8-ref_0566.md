# 10.4 优化数据库结构

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-database-structure.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-database-structure.html)

10.4.1 优化数据大小

10.4.2 优化 MySQL 数据类型

10.4.3 优化多表操作

10.4.4 MySQL 中内部临时表的使用

10.4.5 数据库和表数量限制

10.4.6 表大小限制

10.4.7 表列数和行大小限制

作为数据库设计师，要寻找最有效的方式来组织模式、表和列。就像调整应用程序代码一样，你要尽量减少 I/O，将相关项目放在一起，并提前规划，以确保随着数据量的增加性能保持高水平。从高效的数据库设计开始，可以让团队成员更容易编写高性能的应用程序代码，并使数据库在应用程序演变和重写时能够持久存在。
