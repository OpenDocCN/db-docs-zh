# 15.6.1 BEGIN ... END Compound Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/begin-end.html`](https://dev.mysql.com/doc/refman/8.0/en/begin-end.html)

```sql
[*begin_label*:] BEGIN
    [*statement_list*]
END [*end_label*]
```

`BEGIN ... END`语法用于编写复合语句，这些语句可以出现在存储程序（存储过程和函数、触发器和事件）中。复合语句可以包含多个语句，由`BEGIN`和`END`关键字括起来。*`statement_list`*表示一个或多个语句的列表，每个语句以分号(`;`)语句分隔符结尾。*`statement_list`*本身是可选的，因此空复合语句(`BEGIN END`)是合法的。

`BEGIN ... END`块可以嵌套。

使用多个语句需要客户端能够发送包含`；`语句分隔符的语句字符串。在**mysql**命令行客户端中，可以使用`delimiter`命令处理这个问题。更改`；`语句结束分隔符（例如，改为`//`）允许在程序体中使用`；`。例如，请参阅 Section 27.1, “Defining Stored Programs”。

可以为`BEGIN ... END`块加标签。请参阅 Section 15.6.2, “Statement Labels”。

不支持可选的`[NOT] ATOMIC`子句。这意味着在指令块开始时不设置事务保存点，并且在此上下文中使用的`BEGIN`子句对当前事务没有影响。

注意

在所有存储程序中，解析器将[`BEGIN [WORK]`](commit.html "15.3.1 START TRANSACTION, COMMIT, and ROLLBACK Statements")视为`BEGIN ... END`块的开始。在这种情况下开始事务，请使用`START TRANSACTION`。
