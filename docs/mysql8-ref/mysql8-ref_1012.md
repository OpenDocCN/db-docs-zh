# 15.6.4 存储程序中的变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/stored-program-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/stored-program-variables.html)

15.6.4.1 Local Variable DECLARE Statement

15.6.4.2 本地变量作用域和解析

系统变量和用户定义变量可以在存储程序中使用，就像在存储程序上下文之外使用一样。此外，存储程序可以使用`DECLARE`来定义本地变量，并且存储例程（过程和函数）可以声明接受参数，以在例程和其调用者之间传递值。

+   要声明本地变量，请使用`DECLARE`语句，如 Section 15.6.4.1, “Local Variable DECLARE Statement”中所述。

+   变量可以直接使用`SET`语句进行设置。参见 Section 15.7.6.1, “SET Syntax for Variable Assignment”。

+   查询结果可以使用`SELECT ... INTO *`var_list`*`将其检索到本地变量中，或者通过打开游标并使用`FETCH ... INTO *`var_list`*`来实现。参见 Section 15.2.13.1, “SELECT ... INTO Statement”，以及 Section 15.6.6, “Cursors”。

有关本地变量的作用域以及 MySQL 如何解析模糊名称的信息，请参见 Section 15.6.4.2, “Local Variable Scope and Resolution”。

不允许将值`DEFAULT`分配给存储过程或函数参数或存储程序本地变量（例如使用`SET *`var_name`* = DEFAULT`语句）。在 MySQL 8.0 中，这将导致语法错误。
