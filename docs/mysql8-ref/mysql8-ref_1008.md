# 15.6 复合语句语法

> 原文：[`dev.mysql.com/doc/refman/8.0/en/sql-compound-statements.html`](https://dev.mysql.com/doc/refman/8.0/en/sql-compound-statements.html)

15.6.1 BEGIN ... END 复合语句

15.6.2 语句标签

15.6.3 DECLARE 语句

15.6.4 存储程序中的变量

15.6.5 流程控制语句

15.6.6 游标

15.6.7 条件处理

15.6.8 条件处理的限制

本节描述了`BEGIN ... END`复合语句和其他可以在存储程序主体中使用的语句的语法：存储过程和函数、触发器和事件。这些对象是以 SQL 代码的形式定义的，存储在服务器上以供以后调用（参见第二十七章，*存储对象*）。

复合语句是一个可以包含其他块的块；变量声明、条件处理程序和游标；以及循环和条件测试等流程控制结构。
