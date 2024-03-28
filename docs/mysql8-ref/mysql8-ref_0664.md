# 11.4 用户定义变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/user-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/user-variables.html)

您可以在一个语句中将值存储在用户定义变量中，并在另一个语句中引用它。这使您可以从一个语句传递值到另一个语句。

用户变量写作`@*`var_name`*`，其中变量名*`var_name`*由字母数字字符、`.`、`_`和`$`组成。如果要包含其他字符，可以将其引用为字符串或标识符（例如，`@'my-var'`、`@"my-var"`或`@`my-var``）。

用户定义变量是会话特定的。一个客户端定义的用户变量对其他客户端不可见或不可用。（例外：具有对性能模式`user_variables_by_thread`表的访问权限的用户可以看到所有会话的所有用户变量。）给定客户端会话的所有变量在该客户端退出时会自动释放。

用户变量名称不区分大小写。名称最大长度为 64 个字符。

设置用户定义变量的一种方法是发出`SET`语句：

```sql
SET @*var_name* = *expr* [, @*var_name* = *expr*] ...
```

对于`SET`，赋值操作符可以使用`=`或`:=`。

用户变量可以被赋予整数、小数、浮点、二进制或非二进制字符串、或`NULL`值等有限数据类型的值。赋予小数和实数值不会保留值的精度或标度。除了允许的类型之外的类型的值会被转换为允许的类型。例如，具有时间或空间数据类型的值会被转换为二进制字符串。具有`JSON`数据类型的值会被转换为具有`utf8mb4`字符集和`utf8mb4_bin`校对的字符串。

如果用户变量被赋予非二进制（字符）字符串值，则具有与字符串相同的字符集和校对。用户变量的可强制性是隐式的。（这与表列值的可强制性相同。）

赋予用户变量的十六进制或位值被视为二进制字符串。要将十六进制或位值作为数字赋予用户变量，需在数值上下文中使用。例如，添加 0 或使用`CAST(... AS UNSIGNED)`：

```sql
mysql> SET @v1 = X'41';
mysql> SET @v2 = X'41'+0;
mysql> SET @v3 = CAST(X'41' AS UNSIGNED);
mysql> SELECT @v1, @v2, @v3;
+------+------+------+
| @v1  | @v2  | @v3  |
+------+------+------+
| A    |   65 |   65 |
+------+------+------+
mysql> SET @v1 = b'1000001';
mysql> SET @v2 = b'1000001'+0;
mysql> SET @v3 = CAST(b'1000001' AS UNSIGNED);
mysql> SELECT @v1, @v2, @v3;
+------+------+------+
| @v1  | @v2  | @v3  |
+------+------+------+
| A    |   65 |   65 |
+------+------+------+
```

如果用户变量的值在结果集中被选中，则作为字符串返回给客户端。

如果引用未初始化的变量，则其值为`NULL`，类型为字符串。

从 MySQL 8.0.22 开始，在准备语句中引用用户变量时，其类型在首次准备语句时确定，并在此后每次执行语句时保留此类型。类似地，在存储过程中的语句中使用的用户变量的类型在首次调用存储过程时确定，并在每次后续调用时保留此类型。

用户变量可以在大多数允许表达式的上下文中使用。目前不包括明确要求字面值的上下文，例如`SELECT`语句的`LIMIT`子句，或`LOAD DATA`语句的`IGNORE *`N`* LINES`子句。

MySQL 的先前版本允许在`SET`之外的语句中为用户变量赋值。这种功能在 MySQL 8.0 中得到支持以保持向后兼容性，但可能在将来的 MySQL 版本中被移除。

在这种方式进行赋值时，必须使用`:=`作为赋值运算符；`=`在`SET`之外的语句中被视为比较运算符。

表达式中涉及用户变量的评估顺序是未定义的。例如，不能保证`SELECT @a, @a:=@a+1`首先评估`@a`然后执行赋值。

此外，变量的默认结果类型基于语句开始时的类型。如果变量在语句开始时保存一个类型的值，并在其中分配一个不同类型的新值，则可能会产生意外效果。

为避免此行为问题，在单个语句中不要为同一变量分配值并读取该值，或者在使用之前将变量设置为`0`、`0.0`或`''`以定义其类型。

当在`HAVING`、`GROUP BY`和`ORDER BY`中引用在选择表达式列表中分配值的变量时，不会按预期工作，因为表达式在客户端上评估，因此可能使用来自先前行的过时列值。

用户变量旨在提供数据值。它们不能直接用作 SQL 语句中的标识符或标识符的一部分，例如在期望表或数据库名称的上下文中，或作为保留字，比如`SELECT`。即使变量被引用，也是如此，如下例所示：

```sql
mysql> SELECT c1 FROM t;
+----+
| c1 |
+----+
|  0 |
+----+
|  1 |
+----+
2 rows in set (0.00 sec)

mysql> SET @col = "c1";
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @col FROM t;
+------+
| @col |
+------+
| c1   |
+------+
1 row in set (0.00 sec)

mysql> SELECT `@col` FROM t;
ERROR 1054 (42S22): Unknown column '@col' in 'field list' 
mysql> SET @col = "`c1`";
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @col FROM t;
+------+
| @col |
+------+
| `c1` |
+------+
1 row in set (0.00 sec)
```

不能使用用户变量提供标识符的原则有一个例外，那就是当您构建一个字符串以供稍后执行的准备语句时。在这种情况下，用户变量可以用来提供语句的任何部分。以下示例说明了如何实现这一点：

```sql
mysql> SET @c = "c1";
Query OK, 0 rows affected (0.00 sec)

mysql> SET @s = CONCAT("SELECT ", @c, " FROM t");
Query OK, 0 rows affected (0.00 sec)

mysql> PREPARE stmt FROM @s;
Query OK, 0 rows affected (0.04 sec)
Statement prepared

mysql> EXECUTE stmt;
+----+
| c1 |
+----+
|  0 |
+----+
|  1 |
+----+
2 rows in set (0.00 sec)

mysql> DEALLOCATE PREPARE stmt;
Query OK, 0 rows affected (0.00 sec)
```

更多信息请参见第 15.5 节，“准备语句”。

类似的技术可以在应用程序中使用，以使用程序变量构建 SQL 语句，如下所示使用 PHP 5：

```sql
<?php
  $mysqli = new mysqli("localhost", "user", "pass", "test");

  if( mysqli_connect_errno() )
    die("Connection failed: %s\n", mysqli_connect_error());

  $col = "c1";

  $query = "SELECT $col FROM t";

  $result = $mysqli->query($query);

  while($row = $result->fetch_assoc())
  {
    echo "<p>" . $row["$col"] . "</p>\n";
  }

  $result->close();

  $mysqli->close();
?>
```

以这种方式组装 SQL 语句有时被称为“动态 SQL”。
