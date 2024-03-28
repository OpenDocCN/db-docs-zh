# 15.2.1 CALL 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/call.html`](https://dev.mysql.com/doc/refman/8.0/en/call.html)

```sql
CALL *sp_name*([*parameter*[,...]])
CALL *sp_name*[()]
```

`CALL` 语句调用之前使用 `CREATE PROCEDURE` 之前定义的存储过程。

不带参数的存储过程可以在不使用括号的情况下调用。也就是说，`CALL p()` 和 `CALL p` 是等效的。

`CALL` 可以通过声明为 `OUT` 或 `INOUT` 参数的参数将值传回给调用者。当过程返回时，客户端程序还可以获取在例程内执行的最终语句影响的行数：在 SQL 级别，调用 `ROW_COUNT()` 函数；从 C API，调用 `mysql_affected_rows()` 函数。

有关未处理条件对过程参数的影响的信息，请参见 第 15.6.7.8 节，“条件处理和 OUT 或 INOUT 参数”。

要通过 `OUT` 或 `INOUT` 参数从过程中获取一个值，需要通过用户变量传递参数，然后在过程返回后检查变量的值。（如果你是从另一个存储过程或函数中调用该过程，也可以将例程参数或本地例程变量作为 `IN` 或 `INOUT` 参数传递。）对于 `INOUT` 参数，在传递给过程之前初始化其值。以下过程具有一个 `OUT` 参数，该过程将该参数设置为当前服务器版本，并具有一个 `INOUT` 值，该过程将该值从其当前值增加一：

```sql
DELIMITER //

CREATE PROCEDURE p (OUT ver_param VARCHAR(25), INOUT incr_param INT)
BEGIN
  # Set value of OUT parameter
  SELECT VERSION() INTO ver_param;
  # Increment value of INOUT parameter
  SET incr_param = incr_param + 1;
END //

DELIMITER ;
```

在调用过程之前，初始化要作为 `INOUT` 参数传递的变量。调用过程后，您可以看到这两个变量的值已设置或修改：

```sql
mysql> SET @increment = 10;
mysql> CALL p(@version, @increment);
mysql> SELECT @version, @increment;
+----------+------------+
| @version | @increment |
+----------+------------+
| 8.0.36   |         11 |
+----------+------------+
```

在使用 `PREPARE` 和 `EXECUTE` 语句准备的语句中，可以使用占位符来表示 `IN` 参数、`OUT` 参数和 `INOUT` 参数。这些类型的参数可以如下使用：

```sql
mysql> SET @increment = 10;
mysql> PREPARE s FROM 'CALL p(?, ?)';
mysql> EXECUTE s USING @version, @increment;
mysql> SELECT @version, @increment;
+----------+------------+
| @version | @increment |
+----------+------------+
| 8.0.36   |         11 |
+----------+------------+
```

要编写使用`CALL` SQL 语句执行生成结果集的存储过程的 C 程序，必须启用`CLIENT_MULTI_RESULTS`标志。这是因为每个`CALL`都会返回一个结果来指示调用状态，除了存储过程内执行的可能返回的任何结果集。如果使用`CALL`执行包含准备语句的任何存储过程，也必须启用`CLIENT_MULTI_RESULTS`。无法确定加载此类存储过程时这些语句是否生成结果集，因此必须假定它们会生成结果集。

当您调用`mysql_real_connect()`时，可以通过显式传递`CLIENT_MULTI_RESULTS`标志或隐式传递`CLIENT_MULTI_STATEMENTS`（也会启用`CLIENT_MULTI_RESULTS`）来启用`CLIENT_MULTI_RESULTS`。`CLIENT_MULTI_RESULTS`默认情况下是启用的。

要处理使用`mysql_query()`或`mysql_real_query()`执行的`CALL`语句的结果，请使用调用`mysql_next_result()`的循环来确定是否还有更多结果。有关示例，请参见多语句执行支持。

C 程序可以使用准备语句接口来执行`CALL`语句并访问`OUT`和`INOUT`参数。这是通过处理`CALL`语句的结果，使用调用`mysql_stmt_next_result()`的循环来确定是否还有更多结果。有关示例，请参见准备 CALL 语句支持。提供 MySQL 接口的语言可以使用准备的`CALL`语句直接检索`OUT`和`INOUT`过程参数。

当存储程序引用的对象的元数据发生更改时，会检测到并在下次执行程序时自动重新解析受影响的语句。有关更多信息，请参见第 10.10.3 节，“准备语句和存储程序的缓存”。
