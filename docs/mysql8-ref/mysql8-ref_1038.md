> 原文：[`dev.mysql.com/doc/refman/8.0/en/conditions-and-parameters.html`](https://dev.mysql.com/doc/refman/8.0/en/conditions-and-parameters.html)

#### 15.6.7.8 条件处理和 OUT 或 INOUT 参数

如果存储过程以未处理的异常退出，则`OUT`和`INOUT`参数的修改值不会传播回调用者。

如果异常由包含`RESIGNAL`语句的`CONTINUE`或`EXIT`处理程序处理，`RESIGNAL`语句的执行会弹出诊断区域栈，从而发出异常信号（即，在进入处理程序之前存在的信息）。如果异常是错误，则`OUT`和`INOUT`参数的值不会传播回调用者。
