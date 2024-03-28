# 27.2.1 存储过程语法

> 原文：[`dev.mysql.com/doc/refman/8.0/en/stored-routines-syntax.html`](https://dev.mysql.com/doc/refman/8.0/en/stored-routines-syntax.html)

存储过程可以是过程或函数。存储过程是使用`CREATE PROCEDURE`和`CREATE FUNCTION`语句创建的（参见 Section 15.1.17, “CREATE PROCEDURE and CREATE FUNCTION Statements”）。通过`CALL`语句调用过程（参见 Section 15.2.1, “CALL Statement”），只能使用输出变量传回值。函数可以像其他函数一样从语句内部调用（即通过调用函数的名称），并且可以返回一个标量值。存储过程的主体可以使用复合语句（参见 Section 15.6, “Compound Statement Syntax”）。

存储过程可以使用`DROP PROCEDURE`和`DROP FUNCTION`语句删除（参见 Section 15.1.29, “DROP PROCEDURE and DROP FUNCTION Statements”），并且可以使用`ALTER PROCEDURE`和`ALTER FUNCTION`语句进行修改（参见 Section 15.1.7, “ALTER PROCEDURE Statement”）。

存储过程或函数与特定数据库相关联。这有几个含义：

+   当调用例程时，会执行隐式的`USE *`db_name`*`（在例程终止时撤消）。存储过程内部不允许使用`USE`语句。

+   您可以使用数据库名称限定例程名称。这可用于引用不在当前数据库中的例程。例如，要调用与`test`数据库关联的存储过程`p`或函数`f`，可以使用`CALL test.p()`或`test.f()`。

+   当一个数据库被删除时，与之相关的所有存储过程也会被删除。

存储函数不能是递归的。

存储过程中允许递归，但默认情况下被禁用。要启用递归，请将`max_sp_recursion_depth`服务器系统变量设置为大于零的值。存储过程递归会增加线程堆栈空间的需求。如果增加`max_sp_recursion_depth`的值，可能需要通过增加服务器启动时`thread_stack`的值来增加线程堆栈大小。有关更多信息，请参见第 7.1.8 节，“服务器系统变量”。

MySQL 支持一个非常有用的扩展，允许在存储过程中使用常规的`SELECT`语句（即，不使用游标或本地变量）。这种查询的结果集直接发送到客户端。多个`SELECT`语句会生成多个结果集，因此客户端必须使用支持多个结果集的 MySQL 客户端库。这意味着客户端必须使用至少与 MySQL 4.1 版本一样新的客户端库。客户端在连接时还应指定`CLIENT_MULTI_RESULTS`选项。对于 C 程序，可以使用`mysql_real_connect()` C API 函数来实现。请参见 mysql_real_connect()，以及多语句执行支持。

在 MySQL 8.0.22 及更高版本中，存储过程中的语句引用的用户变量在存储过程首次调用时确定其类型，并在此后的每次调用存储过程时保留此类型。
