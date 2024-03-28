# 15.1.17 CREATE PROCEDURE and CREATE FUNCTION Statements

> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-procedure.html`](https://dev.mysql.com/doc/refman/8.0/en/create-procedure.html)

```sql
CREATE
    [DEFINER = *user*]
    PROCEDURE [IF NOT EXISTS] *sp_name* ([*proc_parameter*[,...]])
    [*characteristic* ...] *routine_body*

CREATE
    [DEFINER = *user*]
    FUNCTION [IF NOT EXISTS] *sp_name* ([*func_parameter*[,...]])
    RETURNS *type*
    [*characteristic* ...] *routine_body*

*proc_parameter*:
    [ IN | OUT | INOUT ] *param_name* *type*

*func_parameter*:
    *param_name* *type*

*type*:
    *Any valid MySQL data type* *characteristic*: {
    COMMENT '*string*'
  | LANGUAGE SQL
  | [NOT] DETERMINISTIC
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
}

*routine_body*:
    *Valid SQL routine statement*
```

这些语句用于创建存储例程（存储过程或函数）。也就是说，指定的例程会被服务器识别。默认情况下，存储例程与默认数据库关联。要将例程明确关联到特定数据库，请在创建时指定名称为*`db_name.sp_name`*。

`CREATE FUNCTION`语句在 MySQL 中还用于支持可加载函数。请参阅 Section 15.7.4.1, “CREATE FUNCTION Statement for Loadable Functions”。可加载函数可以被视为外部存储函数。存储函数与可加载函数共享命名空间。有关服务器解释对不同类型函数引用的规则，请参阅 Section 11.2.5, “Function Name Parsing and Resolution”。

要调用存储过程，请使用`CALL`语句（参见 Section 15.2.1, “CALL Statement”）。要调用存储函数，请在表达式中引用它。在表达式评估期间，函数会返回一个值。

`CREATE PROCEDURE`和`CREATE FUNCTION`需要`CREATE ROUTINE`权限。如果存在`DEFINER`子句，则所需的权限取决于*`user`*值，如 Section 27.6, “Stored Object Access Control”中所讨论的。如果启用了二进制日志记录，则`CREATE FUNCTION`可能需要`SUPER`权限，如 Section 27.7, “Stored Program Binary Logging”中所讨论的。

默认情况下，MySQL 自动授予`ALTER ROUTINE`和`EXECUTE`权限给例程创建者。这种行为可以通过禁用`automatic_sp_privileges`系统变量来更改。请参阅 Section 27.2.2, “Stored Routines and MySQL Privileges”。

`DEFINER`和`SQL SECURITY`子句指定在例程执行时检查访问权限时要使用的安全上下文，如本节后面所述。

如果例程名称与内置 SQL 函数的名称相同，在定义例程或稍后调用它时，如果名称和后面的括号之间没有空格，将会发生语法错误。因此，避免使用现有 SQL 函数的名称作为自己的存储过程名称。

`IGNORE_SPACE` SQL 模式适用于内置函数，而不适用于存储过程。无论 `IGNORE_SPACE` 是否启用，存储过程名称后面都可以有空格。

`IF NOT EXISTS` 可以防止在已经存在同名例程时出现错误。从 MySQL 8.0.29 开始，`CREATE FUNCTION` 和 `CREATE PROCEDURE` 都支持这个选项。

如果同名的内置函数已经存在，尝试使用 `CREATE FUNCTION ... IF NOT EXISTS` 创建存储函数会成功，并显示警告指示它与本地函数同名；这与执行相同的 `CREATE FUNCTION` 语句但不指定 `IF NOT EXISTS` 时没有区别。

如果同名的可加载函数已经存在，使用 `IF NOT EXISTS` 尝试创建存储函数会成功并显示警告。这与不指定 `IF NOT EXISTS` 时的情况相同。

更多信息请参阅函数名称解析。

括在括号内的参数列表必须始终存在。如果没有参数，则应使用空参数列表 `()`。参数名称不区分大小写。

每个参数默认为 `IN` 参数。要为参数指定其他方式，请在参数名称之前使用关键字 `OUT` 或 `INOUT`。

注意

仅对 `PROCEDURE` 指定参数为 `IN`、`OUT` 或 `INOUT` 是有效的。对于 `FUNCTION`，参数始终被视为 `IN` 参数。

`IN` 参数将一个值传递给存储过程。存储过程可能会修改该值，但当存储过程返回时，对调用者不可见。`OUT` 参数将一个值从存储过程传递回调用者。在存储过程内部，其初始值为 `NULL`，当存储过程返回时，其值对调用者可见。`INOUT` 参数由调用者初始化，可以被存储过程修改，存储过程所做的任何更改在存储过程返回时对调用者可见。

对于每个`OUT`或`INOUT`参数，在调用过程的`CALL`语句中传递一个用户定义的变量，以便在过程返回时获取其值。如果您从另一个存储过程或函数内部调用该过程，还可以将例程参数或本地例程变量作为`OUT`或`INOUT`参数传递。如果您从触发器内部调用该过程，还可以将`NEW.*col_name*`作为`OUT`或`INOUT`参数传递。

有关未处理条件对过程参数的影响的信息，请参见 Section 15.6.7.8, “Condition Handling and OUT or INOUT Parameters”。

例程参数不能在例程内准备的语句中引用；请参见 Section 27.8, “Restrictions on Stored Programs”。

以下示例显示了一个简单的存储过程，根据国家代码计算出现在`world`数据库的`city`表中的该国家的城市数量。国家代码使用`IN`参数传递，并使用`OUT`参数返回城市计数：

```sql
mysql> delimiter //

mysql> CREATE PROCEDURE citycount (IN country CHAR(3), OUT cities INT)
       BEGIN
         SELECT COUNT(*) INTO cities FROM world.city
         WHERE CountryCode = country;
       END//
Query OK, 0 rows affected (0.01 sec)

mysql> delimiter ;

mysql> CALL citycount('JPN', @cities); -- cities in Japan
Query OK, 1 row affected (0.00 sec)

mysql> SELECT @cities;
+---------+
| @cities |
+---------+
|     248 |
+---------+
1 row in set (0.00 sec)

mysql> CALL citycount('FRA', @cities); -- cities in France
Query OK, 1 row affected (0.00 sec)

mysql> SELECT @cities;
+---------+
| @cities |
+---------+
|      40 |
+---------+
1 row in set (0.00 sec)
```

该示例使用**mysql**客户端`delimiter`命令在定义过程时将语句定界符从`；`更改为`//`。这样，在过程体中使用的`；`定界符将被传递到服务器而不是被**mysql**本身解释。请参见 Section 27.1, “Defining Stored Programs”。

`RETURNS`子句只能为`FUNCTION`指定，对于`FUNCTION`是强制的。它指示函数的返回类型，函数体必须包含一个`RETURN *value*`语句。如果`RETURN`语句返回不同类型的值，则将该值强制转换为正确的类型。例如，如果函数在`RETURNS`子句中指定了`ENUM`或`SET`值，但`RETURN`语句返回一个整数，则从函数返回的值是相应`ENUM`成员或`SET`成员的字符串。

以下示例函数接受一个参数，使用 SQL 函数执行操作，并返回结果。在这种情况下，不需要使用`delimiter`，因为函数定义不包含内部的`；`语句定界符：

```sql
mysql> CREATE FUNCTION hello (s CHAR(20))
mysql> RETURNS CHAR(50) DETERMINISTIC
       RETURN CONCAT('Hello, ',s,'!');
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT hello('world');
+----------------+
| hello('world') |
+----------------+
| Hello, world!  |
+----------------+
1 row in set (0.00 sec)
```

参数类型和函数返回类型可以声明为任何有效的数据类型。如果在`CHARACTER SET`规范之前使用`COLLATE`属性，则可以使用它。

*`routine_body`* 包含一个有效的 SQL 例程语句。这可以是一个简单的语句，比如`SELECT`或`INSERT`，或者使用`BEGIN`和`END`编写的复合语句。复合语句可以包含声明、循环和其他控制结构语句。这些语句的语法在第 15.6 节“复合语句语法”中描述。在实践中，存储函数往往使用复合语句，除非主体由单个`RETURN`语句组成。

MySQL 允许例程包含 DDL 语句，比如`CREATE`和`DROP`。MySQL 也允许存储过程（但不允许存储函数）包含 SQL 事务语句，比如`COMMIT`。存储函数不能包含执行显式或隐式提交或回滚的语句。SQL 标准不要求支持这些语句，它规定每个 DBMS 供应商可以决定是否允许它们。

返回结果集的语句可以在存储过程中使用，但不能在存储函数中使用。这个禁令包括没有`INTO *`var_list`*`子句的`SELECT`语句以及其他语句，比如`SHOW`、`EXPLAIN`和`CHECK TABLE`。对于在函数定义时可以确定返回结果集的语句，会出现`Not allowed to return a result set from a function`错误（`ER_SP_NO_RETSET`）。对于只能在运行时确定返回结果集的语句，会出现`PROCEDURE %s can't return a result set in the given context`错误（`ER_SP_BADSELECT`）。

存储例程中不允许使用`USE`语句。当调用例程时，会执行一个隐式的`USE *`db_name`*`（在例程终止时撤消）。这会导致例程在执行时具有给定的默认数据库。对于例程默认数据库之外的数据库中的对象的引用应该使用适当的数据库名称进行限定。

有关存储例程中不允许的语句的更多信息，请参见第 27.8 节“存储程序的限制”。

有关如何从具有 MySQL 接口的语言编写的程序中调用存储过程的信息，请参见第 15.2.1 节“CALL 语句”。

MySQL 在创建或更改例程时存储 `sql_mode` 系统变量设置，并始终以此设置执行例程，*不管例程开始执行时当前服务器 SQL 模式如何*。

从调用者的 SQL 模式切换到例程的 SQL 模式发生在参数评估和将结果值分配给例程参数之后。如果在严格 SQL 模式下定义例程但在非严格模式下调用它，则参数分配给例程参数不会在严格模式下进行。如果要求传递给例程的表达式在严格 SQL 模式下分配，应该在调用例程时启用严格模式。

`COMMENT` 特性是 MySQL 的扩展，可用于描述存储例程。这些信息会被 `SHOW CREATE PROCEDURE` 和 `SHOW CREATE FUNCTION` 语句显示。

`LANGUAGE` 特性表示例程所编写的语言。服务器会忽略此特性；仅支持 SQL 例程。

如果一个例程对于相同的输入参数总是产生相同的结果，则被认为是“确定性的”，否则是“非确定性的”。如果例程定义中既没有 `DETERMINISTIC` 也没有 `NOT DETERMINISTIC`，则默认为 `NOT DETERMINISTIC`。要声明一个函数是确定性的，必须明确指定 `DETERMINISTIC`。

对例程性质的评估基于创建者的“诚实度”：MySQL 不会检查声明为 `DETERMINISTIC` 的例程是否不包含产生非确定性结果的语句。然而，错误声明例程可能会影响结果或性能。将一个非确定性例程声明为 `DETERMINISTIC` 可能会导致意外结果，因为优化器会做出错误的执行计划选择。将一个确定性例程声明为 `NONDETERMINISTIC` 可能会降低性能，因为可用的优化不会被使用。

如果启用了二进制日志记录，`DETERMINISTIC` 特性会影响 MySQL 接受哪些例程定义。参见 第 27.7 节，“存储程序二进制日志记录”。

包含`NOW()`函数（或其同义词）或`RAND()`的例程是不确定性的，但可能仍然是复制安全的。对于`NOW()`，二进制日志包括时间戳并正确复制。`RAND()`只要在例程执行过程中仅调用一次，也会正确复制。（您可以将例程执行时间戳和随机数种子视为在源和副本上相同的隐式输入。）

几个特性提供有关例程使用数据性质的信息。在 MySQL 中，这些特性仅供参考。服务器不使用它们来限制例程允许执行的语句类型。

+   `CONTAINS SQL`表示例程不包含读取或写入数据的语句。如果没有明确给出这些特性中的任何一个，则这是默认值。此类语句的示例是`SET @x = 1`或`DO RELEASE_LOCK('abc')`，它们执行但既不读取也不写入数据。

+   `NO SQL`表示例程不包含 SQL 语句。

+   `READS SQL DATA`表示例程包含读取数据的语句（例如，`SELECT`），但不包含写入数据的语句。

+   `MODIFIES SQL DATA`表示例程包含可能写入数据的语句（例如，`INSERT`或`DELETE`）。

`SQL SECURITY`特性可以是`DEFINER`或`INVOKER`，用于指定安全上下文；也就是说，例程是使用例程`DEFINER`子句中命名的帐户的权限执行，还是由调用者执行。此帐户必须具有访问与例程关联的数据库的权限。默认值为`DEFINER`。调用例程的用户必须具有执行权限，以及如果例程在定义者安全上下文中执行，则`DEFINER`帐户也必须具有执行权限。

`DEFINER`子句指定了在具有`SQL SECURITY DEFINER`特性的例程执行时检查访问权限时要使用的 MySQL 帐户。

如果存在`DEFINER`子句，则*`user`*值应为指定为`'*`user_name`*'@'*`host_name`*`、`CURRENT_USER`或`CURRENT_USER()`的 MySQL 帐户。允许的*`user`*值取决于您拥有的权限，如第 27.6 节“存储对象访问控制”中所讨论的。此外，请参阅该部分以获取有关存储例程安全性的其他信息。

如果省略`DEFINER`子句，则默认的定义者是执行`CREATE PROCEDURE`或`CREATE FUNCTION`语句的用户。这与明确指定`DEFINER = CURRENT_USER`相同。

在具有`SQL SECURITY DEFINER`特性定义的存储例程体内，`CURRENT_USER`函数返回例程的`DEFINER`值。有关存储例程中用户审计的信息，请参见第 8.2.23 节，“基于 SQL 的帐户活动审计”。

考虑以下过程，该过程显示`mysql.user`系统表中列出的 MySQL 帐户数量的计数：

```sql
CREATE DEFINER = 'admin'@'localhost' PROCEDURE account_count()
BEGIN
  SELECT 'Number of accounts:', COUNT(*) FROM mysql.user;
END;
```

无论哪个用户定义该过程，该过程都被分配了一个`'admin'@'localhost'`的`DEFINER`帐户。它以该帐户的权限执行，无论哪个用户调用它（因为默认的安全特性是`DEFINER`）。该过程的成功或失败取决于调用者是否具有`EXECUTE`权限以及`'admin'@'localhost'`是否具有`mysql.user`表的`SELECT`权限。

现在假设该过程使用`SQL SECURITY INVOKER`特性定义：

```sql
CREATE DEFINER = 'admin'@'localhost' PROCEDURE account_count()
SQL SECURITY INVOKER
BEGIN
  SELECT 'Number of accounts:', COUNT(*) FROM mysql.user;
END;
```

该过程仍然具有`'admin'@'localhost'`的`DEFINER`，但在这种情况下，它以调用用户的权限执行。因此，该过程的成功或失败取决于调用者是否具有`EXECUTE`权限以及`mysql.user`表的`SELECT`权限。

默认情况下，当具有`SQL SECURITY DEFINER`特性的例程被执行时，MySQL 服务器不会为`DEFINER`子句中命名的 MySQL 帐户设置任何活动角色，只有默认角色。例外情况是如果启用了`activate_all_roles_on_login`系统变量，此时 MySQL 服务器会设置授予`DEFINER`用户的所有角色，包括强制角色。因此，默认情况下，在发出`CREATE PROCEDURE`或`CREATE FUNCTION`语句时，不会检查通过角色授予的任何权限。对于存储程序，如果执行应该使用与默认不同的角色，则程序体可以执行`SET ROLE`来激活所需的角色。这必须谨慎进行，因为分配给角色的权限可能会更改。

服务器处理例程参数、使用`DECLARE`创建的本地例程变量，或函数返回值的数据类型如下：

+   分配会检查数据类型不匹配和溢出。转换和溢出问题会导致警告，或在严格的 SQL 模式下出现错误。

+   只能分配标量值。例如，`SET x = (SELECT 1, 2)`这样的语句是无效的。

+   对于字符数据类型，如果声明中包含`CHARACTER SET`，则使用指定的字符集及其默认排序规则。如果还存在`COLLATE`属性，则使用该排序规则而不是默认排序规则。

    如果没有`CHARACTER SET`和`COLLATE`，则在例程创建时生效的数据库字符集和排序规则会被使用。为了避免服务器使用数据库字符集和排序规则，请为字符数据参数提供明确的`CHARACTER SET`和`COLLATE`属性。

    如果更改数据库默认字符集或排序规则，则必须删除并重新创建要使用新数据库默认值的存储例程。

    数据库字符集和排序规则由`character_set_database`和`collation_database`系统变量的值给出。更多信息，请参见 Section 12.3.3, “Database Character Set and Collation”。
