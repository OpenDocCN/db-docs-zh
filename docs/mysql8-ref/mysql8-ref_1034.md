> 原文：[`dev.mysql.com/doc/refman/8.0/en/resignal.html`](https://dev.mysql.com/doc/refman/8.0/en/resignal.html)

#### 15.6.7.4 RESIGNAL 语句

```sql
RESIGNAL [*condition_value*]
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

`RESIGNAL` 传递在存储过程、函数、触发器或事件内部的复合语句中执行条件处理程序期间可用的错误条件信息。`RESIGNAL` 可能在传递信息之前更改部分或全部信息。`RESIGNAL` 与 `SIGNAL` 相关，但与 `SIGNAL` 不同，`RESIGNAL` 转发现有的条件信息，可能在修改后传递。

`RESIGNAL` 使得处理错误并返回错误信息成为可能。否则，在处理程序内执行 SQL 语句时，导致处理程序激活的信息将被销毁。`RESIGNAL` 还可以使一些过程变得更短，如果给定的处理程序可以处理部分情况，然后将条件“传递给上一级”到另一个处理程序。

执行 `RESIGNAL` 语句不需要特权。

所有形式的 `RESIGNAL` 需要当前上下文为条件处理程序。否则，`RESIGNAL` 是非法的，会出现 `RESIGNAL when handler not active` 错误。

要从诊断区域检索信息，请使用 `GET DIAGNOSTICS` 语句（参见 Section 15.6.7.3, “GET DIAGNOSTICS Statement”）。有关诊断区域的信息，请参阅 Section 15.6.7.7, “The MySQL Diagnostics Area”。

+   RESIGNAL 概述

+   单独使用 RESIGNAL

+   带有新信号信息的 RESIGNAL

+   带有条件值和可选新信号信息的 RESIGNAL

+   RESIGNAL 需要条件处理程序上下文

##### RESIGNAL 概述

对于*`condition_value`*和*`signal_information_item`*，`RESIGNAL`的定义和规则与`SIGNAL`相同。例如，*`condition_value`*可以是`SQLSTATE`值，该值可以指示错误、警告或“未找到”。有关更多信息，请参阅第 15.6.7.5 节，“SIGNAL Statement”。

`RESIGNAL`语句接受*`condition_value`*和`SET`子句，两者都是可选的。这导致了几种可能的用法：

+   单独的`RESIGNAL`：

    ```sql
    RESIGNAL;
    ```

+   `RESIGNAL`带有新的信号信息：

    ```sql
    RESIGNAL SET *signal_information_item* [, *signal_information_item*] ...;
    ```

+   带有条件值和可能的新信号信息的`RESIGNAL`：

    ```sql
    RESIGNAL *condition_value*
        [SET *signal_information_item* [, *signal_information_item*] ...];
    ```

这些用例都会导致诊断和条件区域的更改：

+   诊断区包含一个或多个条件区域。

+   条件区域包含条件信息项，例如`SQLSTATE`值、`MYSQL_ERRNO`或`MESSAGE_TEXT`。

有一个诊断区堆栈。当处理程序控制时，它会将诊断区推送到堆栈顶部，因此在处理程序执行期间有两个诊断区：

+   第一个（当前）诊断区，最初是最后一个诊断区的副本，但会被第一个改变当前诊断区的处理程序中的第一个语句覆盖。

+   最后一个（堆叠的）诊断区，其中包含在处理程序控制之前设置的条件区域。

诊断区中条件区域的最大数量由`max_error_count`系统变量的值确定。请参阅诊断区相关系统变量。

##### 单独的 RESIGNAL

简单的`RESIGNAL`单独表示“不做任何更改地传递错误”。它恢复最后的诊断区并将其设置为当前诊断区。也就是说，它“弹出”诊断区堆栈。

在捕获条件的条件处理程序中，`RESIGNAL`单独的一个用法是执行其他操作，然后在不改变原始条件信息的情况下继续传递（即在进入处理程序之前存在的信息）。

例子：

```sql
DROP TABLE IF EXISTS xx;
delimiter //
CREATE PROCEDURE p ()
BEGIN
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    SET @error_count = @error_count + 1;
    IF @a = 0 THEN RESIGNAL; END IF;
  END;
  DROP TABLE xx;
END//
delimiter ;
SET @error_count = 0;
SET @a = 0;
CALL p();
```

假设`DROP TABLE xx`语句失败。诊断区堆栈如下：

```sql
DA 1\. ERROR 1051 (42S02): Unknown table 'xx'
```

然后执行进入`EXIT`处理程序。它开始通过将诊断区推送到堆栈顶部，现在看起来像这样：

```sql
DA 1\. ERROR 1051 (42S02): Unknown table 'xx'
DA 2\. ERROR 1051 (42S02): Unknown table 'xx'
```

此时，第一个（当前）和第二个（堆叠的）诊断区的内容相同。第一个诊断区可能会被在处理程序内随后执行的语句修改。

通常，过程语句会清除第一个诊断区。`BEGIN`是一个例外，它不清除，什么也不做。`SET`不是例外，它会清除，执行操作，并产生“成功”的结果。诊断区堆栈现在看��来像这样：

```sql
DA 1\. ERROR 0000 (00000): Successful operation
DA 2\. ERROR 1051 (42S02): Unknown table 'xx'
```

在这一点上，如果`@a = 0`，`RESIGNAL`会弹出诊断区堆栈，现在看起来像这样：

```sql
DA 1\. ERROR 1051 (42S02): Unknown table 'xx'
```

这就是调用者看到的。

如果`@a`不等于 0，处理程序简单地结束，这意味着当前诊断区不再有用（已经被“处理”），因此可以丢弃它，导致堆叠的诊断区变成当前诊断区。诊断区堆栈看起来像这样：

```sql
DA 1\. ERROR 0000 (00000): Successful operation
```

细节使其看起来复杂，但最终结果非常有用：处理程序可以执行而不破坏导致处理程序激活的条件的信息。

##### 带有新信号信息的 RESIGNAL

带有`SET`子句的`RESIGNAL`提供新的信号信息，因此该语句意味着“传递带有更改的错误”：

```sql
RESIGNAL SET *signal_information_item* [, *signal_information_item*] ...;
```

与单独使用`RESIGNAL`一样，其思想是弹出诊断区堆栈，以便原始信息被清除。与单独使用`RESIGNAL`不同的是，`SET`子句中指定的任何内容都会发生变化。

例子：

```sql
DROP TABLE IF EXISTS xx;
delimiter //
CREATE PROCEDURE p ()
BEGIN
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    SET @error_count = @error_count + 1;
    IF @a = 0 THEN RESIGNAL SET MYSQL_ERRNO = 5; END IF;
  END;
  DROP TABLE xx;
END//
delimiter ;
SET @error_count = 0;
SET @a = 0;
CALL p();
```

从之前的讨论中记得，单独使用`RESIGNAL`会导致诊断区堆栈如下：

```sql
DA 1\. ERROR 1051 (42S02): Unknown table 'xx'
```

`RESIGNAL SET MYSQL_ERRNO = 5`语句导致了这个堆栈，这是调用者看到的：

```sql
DA 1\. ERROR 5 (42S02): Unknown table 'xx'
```

换句话说，它改变了错误编号，而不会改变其他任何东西。

`RESIGNAL`语句可以更改任何或所有信号信息项，使诊断区的第一个条件区域看起来完全不同。

##### 带有条件值和可选新信号信息的 RESIGNAL

带有条件值的`RESIGNAL`意味着“将一个条件推入当前诊断区。”如果存在`SET`子句，它还会改变错误信息。

```sql
RESIGNAL *condition_value*
    [SET *signal_information_item* [, *signal_information_item*] ...];
```

这种形式的`RESIGNAL`恢复了最后的诊断区并将其作为当前诊断区。也就是说，它“弹出”了诊断区堆栈，这与简单使用`RESIGNAL`的效果相同。但是，它还会根据条件值或信号信息更改诊断区。

例子：

```sql
DROP TABLE IF EXISTS xx;
delimiter //
CREATE PROCEDURE p ()
BEGIN
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    SET @error_count = @error_count + 1;
    IF @a = 0 THEN RESIGNAL SQLSTATE '45000' SET MYSQL_ERRNO=5; END IF;
  END;
  DROP TABLE xx;
END//
delimiter ;
SET @error_count = 0;
SET @a = 0;
SET @@max_error_count = 2;
CALL p();
SHOW ERRORS;
```

这与之前的例子类似，效果相同，只是如果发生`RESIGNAL`，则最终当前条件区域看起来不同。（条件增加而不是替换现有条件的原因是使用了条件值。）

`RESIGNAL` 语句包括一个条件值（`SQLSTATE '45000'`），因此它添加了一个新的条件区域，导致诊断区域堆栈如下所示：

```sql
DA 1\. (condition 2) ERROR 1051 (42S02): Unknown table 'xx'
      (condition 1) ERROR 5 (45000) Unknown table 'xx'
```

对于这个例子，`CALL p()` 和 `SHOW ERRORS` 的结果是：

```sql
mysql> CALL p();
ERROR 5 (45000): Unknown table 'xx'
mysql> SHOW ERRORS;
+-------+------+----------------------------------+
| Level | Code | Message                          |
+-------+------+----------------------------------+
| Error | 1051 | Unknown table 'xx'               |
| Error |    5 | Unknown table 'xx'               |
+-------+------+----------------------------------+
```

##### RESIGNAL 需要条件处理程序上下文

所有形式的`RESIGNAL`都要求当前上下文是一个条件处理程序。否则，`RESIGNAL`是非法的，会出现`RESIGNAL when handler not active`错误。例如：

```sql
mysql> CREATE PROCEDURE p () RESIGNAL;
Query OK, 0 rows affected (0.00 sec)

mysql> CALL p();
ERROR 1645 (0K000): RESIGNAL when handler not active
```

这里是一个更加困难的例子：

```sql
delimiter //
CREATE FUNCTION f () RETURNS INT
BEGIN
  RESIGNAL;
  RETURN 5;
END//
CREATE PROCEDURE p ()
BEGIN
  DECLARE EXIT HANDLER FOR SQLEXCEPTION SET @a=f();
  SIGNAL SQLSTATE '55555';
END//
delimiter ;
CALL p();
```

`RESIGNAL` 发生在存储函数 `f()` 中。虽然 `f()` 本身是在 `EXIT` 处理程序的上下文中调用的，但在 `f()` 中的执行有其自己的上下文，这不是处理程序上下文。因此，在 `f()` 中的 `RESIGNAL` 导致“处理程序未激活”错误。
