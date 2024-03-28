> 原文：[`dev.mysql.com/doc/refman/8.0/en/signal.html`](https://dev.mysql.com/doc/refman/8.0/en/signal.html)

#### 15.6.7.5 信号语句

```sql
SIGNAL *condition_value*
    [SET *signal_information_item*
    [, *signal_information_item*] ...]

*condition_value*: {
    SQLSTATE [VALUE] *sqlstate_value*
  | *condition_name*
}

*signal_information_item*:
    *condition_information_item_name* = *simple_value_specification*

*condition_information_item_name*: {
    CLASS_ORIGIN
  | SUBCLASS_ORIGIN
  | MESSAGE_TEXT
  | MYSQL_ERRNO
  | CONSTRAINT_CATALOG
  | CONSTRAINT_SCHEMA
  | CONSTRAINT_NAME
  | CATALOG_NAME
  | SCHEMA_NAME
  | TABLE_NAME
  | COLUMN_NAME
  | CURSOR_NAME
}

*condition_name*, *simple_value_specification*:
    (see following discussion)
```

`SIGNAL`是“返回”错误的方法。`SIGNAL`向处理程序、应用程序的外部部分或客户端提供错误信息。此外，它还可以控制错误的特性（错误编号、`SQLSTATE`值、消息）。没有`SIGNAL`，就必须诉诸解决方法，例如故意引用一个不存在的表来导致例程返回错误。

执行`SIGNAL`语句不需要特权。

要从诊断区域检索信息，请使用`GET DIAGNOSTICS`语句（参见第 15.6.7.3 节，“GET DIAGNOSTICS 语句”）。有关诊断区域的信息，请参阅第 15.6.7.7 节，“MySQL 诊断区域”。

+   信号概述

+   信号条件信息项

+   信号对处理程序、游标和语句的影响

##### 信号概述

在`SIGNAL`语句中的*`condition_value`*表示要返回的错误值。它可以是一个`SQLSTATE`值（一个 5 个字符的字符串文字）或一个*`condition_name`*，它引用先前使用`DECLARE ... CONDITION`定义的命名条件（参见第 15.6.7.1 节，“DECLARE ... CONDITION 语句”）。

一个`SQLSTATE`值可以指示错误、警告或“未找到”。该值的前两个字符表示其错误类别，如信号条件信息项中所讨论的那样。一些信号值会导致语句终止；请参阅信号对处理程序、游标和语句的影响。

`SIGNAL`语句的`SQLSTATE`值不应以`'00'`开头，因为这样的值表示成功，不适用于发出错误信号。无论`SQLSTATE`值是直接在`SIGNAL`语句中指定还是在语句中引用的命名条件中引用，都是如此。如果该值无效，则会发生`Bad SQLSTATE`错误。

要发出通用的`SQLSTATE`值，请使用`'45000'`，表示“未处理的用户定义异常”。

`SIGNAL`语句可选地包含一个`SET`子句，其中包含多个信号项，以*`condition_information_item_name`* = *`simple_value_specification`*分配的列表，用逗号分隔。

每个*`condition_information_item_name`*在`SET`子句中只能指定一次。否则，会出现`Duplicate condition information item`错误。

可以使用存储过程或函数参数、使用`DECLARE`声明的存储程序本地变量、用户定义变量、系统变量或文字指定有效的*`simple_value_specification`*标识符。字符文字可能包括一个*`_charset`*引导符。

有关可接受的*`condition_information_item_name`*值的信息，请参阅 Signal Condition Information Items。

以下过程根据其输入参数`pval`的值发出错误或警告：

```sql
CREATE PROCEDURE p (pval INT)
BEGIN
  DECLARE specialty CONDITION FOR SQLSTATE '45000';
  IF pval = 0 THEN
    SIGNAL SQLSTATE '01000';
  ELSEIF pval = 1 THEN
    SIGNAL SQLSTATE '45000'
      SET MESSAGE_TEXT = 'An error occurred';
  ELSEIF pval = 2 THEN
    SIGNAL specialty
      SET MESSAGE_TEXT = 'An error occurred';
  ELSE
    SIGNAL SQLSTATE '01000'
      SET MESSAGE_TEXT = 'A warning occurred', MYSQL_ERRNO = 1000;
    SIGNAL SQLSTATE '45000'
      SET MESSAGE_TEXT = 'An error occurred', MYSQL_ERRNO = 1001;
  END IF;
END;
```

如果`pval`为 0，`p()`会发出一个警告，因为以`'01'`开头的`SQLSTATE`值属于警告类别。警告不会终止该过程，并且在过程返回后可以使用`SHOW WARNINGS`查看。

如果`pval`为 1，`p()`会发出一个错误并设置`MESSAGE_TEXT`条件信息项。错误会终止该过程，并且文本将随错误信息一起返回。

如果`pval`为 2，则会发出相同的错误，尽管在这种情况下使用命名条件指定了`SQLSTATE`值。

如果`pval`为其他任何值，`p()`首先发出一个警告并设置消息文本和错误编号条件信息项。此警告不会终止该过程，因此执行会继续，然后`p()`会发出一个错误。错误会终止该过程。警告设置的消息文本和错误编号将被错误设置的值替换，这些值将与错误信息一起返回。

`SIGNAL`通常在存储程序中使用，但是 MySQL 扩展允许在处理程序上下文之外使用。例如，如果调用**mysql**客户端程序，则可以在提示符下输入以下任何语句：

```sql
SIGNAL SQLSTATE '77777';

CREATE TRIGGER t_bi BEFORE INSERT ON t
  FOR EACH ROW SIGNAL SQLSTATE '77777';

CREATE EVENT e ON SCHEDULE EVERY 1 SECOND
  DO SIGNAL SQLSTATE '77777';
```

`SIGNAL`根据以下规则执行：

如果`SIGNAL`语句指示特定的`SQLSTATE`值，则该值用于发出指定的条件。例如：

```sql
CREATE PROCEDURE p (divisor INT)
BEGIN
  IF divisor = 0 THEN
    SIGNAL SQLSTATE '22012';
  END IF;
END;
```

如果 `SIGNAL` 语句使用了一个命名条件，该条件必须在适用于 `SIGNAL` 语句的某个范围内声明，并且必须使用 `SQLSTATE` 值而不是 MySQL 错误编号进行定义。示例：

```sql
CREATE PROCEDURE p (divisor INT)
BEGIN
  DECLARE divide_by_zero CONDITION FOR SQLSTATE '22012';
  IF divisor = 0 THEN
    SIGNAL divide_by_zero;
  END IF;
END;
```

如果在 `SIGNAL` 语句的范围内不存在命名条件，则会发生 `Undefined CONDITION` 错误。

如果 `SIGNAL` 引用了一个使用 MySQL 错误编号而不是 `SQLSTATE` 值定义的命名条件，则会发生 `SIGNAL/RESIGNAL 只能使用使用 SQLSTATE 定义的 CONDITION` 错误。以下语句会导致该错误，因为命名条件与 MySQL 错误编号相关联：

```sql
DECLARE no_such_table CONDITION FOR 1051;
SIGNAL no_such_table;
```

如果在不同范围内多次声明具有相同名称的条件，则具有最局部范围的声明适用。考虑以下过程：

```sql
CREATE PROCEDURE p (divisor INT)
BEGIN
  DECLARE my_error CONDITION FOR SQLSTATE '45000';
  IF divisor = 0 THEN
    BEGIN
      DECLARE my_error CONDITION FOR SQLSTATE '22012';
      SIGNAL my_error;
    END;
  END IF;
  SIGNAL my_error;
END;
```

如果 `divisor` 为 0，则执行第一个 `SIGNAL` 语句。最内层的 `my_error` 条件声明适用，引发 `SQLSTATE` `'22012'`。

如果 `divisor` 不为 0，则执行第二个 `SIGNAL` 语句。最外层的 `my_error` 条件声明适用，引发 `SQLSTATE` `'45000'`。

有关服务器在发生条件时选择处理程序的信息，请参阅 第 15.6.7.6 节，“处理程序的作用域规则”。

异常处理程序内部可以引发信号：

```sql
CREATE PROCEDURE p ()
BEGIN
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    SIGNAL SQLSTATE VALUE '99999'
      SET MESSAGE_TEXT = 'An error occurred';
  END;
  DROP TABLE no_such_table;
END;
```

`CALL p()` 到达 `DROP TABLE` 语句。没有名为 `no_such_table` 的表，因此激活了错误处理程序。错误处理程序销毁原始错误（“没有这样的表”），并生成一个具有 `SQLSTATE` `'99999'` 和消息 `An error occurred` 的新错误。

##### 信号条件信息项

以下表列出了可以在 `SIGNAL`（或 `RESIGNAL`）语句中设置的诊断区域条件信息项的名称。所有项目都是标准 SQL，除了 `MYSQL_ERRNO`，它是 MySQL 的扩展。有关这些项目的更多信息，请参阅 第 15.6.7.7 节，“MySQL 诊断区域”。

```sql
Item Name             Definition
---------             ----------
CLASS_ORIGIN          VARCHAR(64)
SUBCLASS_ORIGIN       VARCHAR(64)
CONSTRAINT_CATALOG    VARCHAR(64)
CONSTRAINT_SCHEMA     VARCHAR(64)
CONSTRAINT_NAME       VARCHAR(64)
CATALOG_NAME          VARCHAR(64)
SCHEMA_NAME           VARCHAR(64)
TABLE_NAME            VARCHAR(64)
COLUMN_NAME           VARCHAR(64)
CURSOR_NAME           VARCHAR(64)
MESSAGE_TEXT          VARCHAR(128)
MYSQL_ERRNO           SMALLINT UNSIGNED
```

字符项的字符集为 UTF-8。

在 `SIGNAL` 语句中将 `NULL` 赋给条件信息项是非法的。

`SIGNAL` 语句总是指定一个 `SQLSTATE` 值，可以直接指定，也可以间接引用具有 `SQLSTATE` 值的命名条件。`SQLSTATE` 值的前两个字符是其类别，类别确定条件信息项的默认值：

+   类 = `'00'`（成功）

    非法。以`'00'`开头的`SQLSTATE`值表示成功，并且对于`SIGNAL`无效。

+   类别 = `'01'`（警告）

    ```sql
    MESSAGE_TEXT = 'Unhandled user-defined warning condition';
    MYSQL_ERRNO = ER_SIGNAL_WARN
    ```

+   类别 = `'02'`（未找到）

    ```sql
    MESSAGE_TEXT = 'Unhandled user-defined not found condition';
    MYSQL_ERRNO = ER_SIGNAL_NOT_FOUND
    ```

+   类别 > `'02'`（异常）

    ```sql
    MESSAGE_TEXT = 'Unhandled user-defined exception condition';
    MYSQL_ERRNO = ER_SIGNAL_EXCEPTION
    ```

对于合法类别，其他条件信息项设置如下：

```sql
CLASS_ORIGIN = SUBCLASS_ORIGIN = '';
CONSTRAINT_CATALOG = CONSTRAINT_SCHEMA = CONSTRAINT_NAME = '';
CATALOG_NAME = SCHEMA_NAME = TABLE_NAME = COLUMN_NAME = '';
CURSOR_NAME = '';
```

在`SIGNAL`执行后可访问的错误值是由`SIGNAL`语句引发的`SQLSTATE`值以及`MESSAGE_TEXT`和`MYSQL_ERRNO`项。这些值可从 C API 中获取：

+   `mysql_sqlstate()`返回`SQLSTATE`值。

+   `mysql_errno()`返回`MYSQL_ERRNO`值。

+   `mysql_error()`返回`MESSAGE_TEXT`值。

在 SQL 级别，`SHOW WARNINGS`和`SHOW ERRORS`的输出指示`Code`和`Message`列中的`MYSQL_ERRNO`和`MESSAGE_TEXT`值。

要从诊断区域检索信息，请使用`GET DIAGNOSTICS`语句（参见 Section 15.6.7.3, “GET DIAGNOSTICS Statement”）。有关诊断区域的信息，请参阅 Section 15.6.7.7, “The MySQL Diagnostics Area”。

##### 信号对处理程序、游标和语句的影响

信号对语句执行的影响取决于信号类别。类别确定错误的严重程度。MySQL 忽略`sql_mode`系统变量的值；特别是，严格的 SQL 模式无关紧要。MySQL 还忽略`IGNORE`：`SIGNAL`的目的是明确引发用户生成的错误，因此信号永远不会被忽略。

在以下描述中，“未处理”表示未使用`DECLARE ... HANDLER`为信号的`SQLSTATE`值定义处理程序。

+   类别 = `'00'`（成功）

    非法。以`'00'`开头的`SQLSTATE`值表示成功，并且对于`SIGNAL`无效。

+   类别 = `'01'`（警告）

    `warning_count`系统变量的值增加。`SHOW WARNINGS`显示信号。`SQLWARNING`处理程序捕获信号。

    由于导致函数返回的`RETURN`语句清除了诊断区域，因此无法从存储函数中返回警告。该语句清除了可能存在的任何警告（并将`warning_count`重置为 0）。

+   类别 = `'02'`（未找到）

    `NOT FOUND`处理程序捕获信号。对游标没有影响。如果在存储函数中未处理信号，则语句结束。

+   类别 > `'02'`（异常）

    `SQLEXCEPTION`处理程序捕获信号。如果在存储函数中未处理信号，则语句结束。

+   类别 = `'40'`

    被视为普通异常。
