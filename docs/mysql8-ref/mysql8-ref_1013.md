> 原文：[`dev.mysql.com/doc/refman/8.0/en/declare-local-variable.html`](https://dev.mysql.com/doc/refman/8.0/en/declare-local-variable.html)

#### 15.6.4.1 局部变量 DECLARE 语句

```sql
DECLARE *var_name* [, *var_name*] ... *type* [DEFAULT *value*]
```

此语句在存储程序中声明局部变量。要为变量提供默认值，请包含`DEFAULT`子句。该值可以指定为表达式；它不必是常量。如果缺少`DEFAULT`子句，则初始值为`NULL`。

局部变量在数据类型和溢出检查方面与存储过程参数类似。请参阅第 15.1.17 节，“CREATE PROCEDURE 和 CREATE FUNCTION 语句”。

变量声明必须出现在游标或处理程序声明之前。

局部变量名称不区分大小写。允许的字符和引用规则与其他标识符相同，如第 11.2 节，“模式对象名称”中所述。

局部变量的作用域是其声明的`BEGIN ... END`块。该变量可以在声明块内嵌套的块中引用，除了那些声明具有相同名称变量的块。

有关变量声明的示例，请参阅第 15.6.4.2 节，“局部变量作用域和解析”。
