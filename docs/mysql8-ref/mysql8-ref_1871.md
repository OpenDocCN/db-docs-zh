# 27.2.4 存储过程、函数、触发器和 LAST_INSERT_ID()

> 原文：[`dev.mysql.com/doc/refman/8.0/en/stored-routines-last-insert-id.html`](https://dev.mysql.com/doc/refman/8.0/en/stored-routines-last-insert-id.html)

在存储过程（procedure 或 function）或触发器的主体内，`LAST_INSERT_ID()`的值会像在这些对象的主体外执行语句时一样发生变化（参见第 14.15 节，“信息函数”）。存储过程或触发器对`LAST_INSERT_ID()`值的影响取决于存储过程的类型：

+   如果存储过程执行更改`LAST_INSERT_ID()`值的语句，则后续语句会看到更改后的值。

+   对于改变值的存储函数和触发器，在函数或触发器结束时，值会恢复，因此后续语句不会看到更改后的值。
