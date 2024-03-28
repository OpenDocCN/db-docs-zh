> 原文：[`dev.mysql.com/doc/refman/8.0/en/handler-scope.html`](https://dev.mysql.com/doc/refman/8.0/en/handler-scope.html)

#### 15.6.7.6 处理程序的范围规则

存储过程可以包括在程序内发生某些条件时调用的处理程序。每个处理程序的适用性取决于其在程序定义中的位置以及它处理的条件或条件：

+   在`BEGIN ... END`块中声明的处理程序仅对在块中处理程序声明后的 SQL 语句有效。如果处理程序本身引发条件，则它无法处理该条件，也不能处理块中声明的任何其他处理程序。在下面的示例中，处理程序`H1`和`H2`适用于*`stmt1`*和*`stmt2`*语句引发的条件。但是`H1`和`H2`对于在`H1`或`H2`主体中引发的条件不适用。

    ```sql
    BEGIN -- outer block
      DECLARE EXIT HANDLER FOR ...;  -- handler H1
      DECLARE EXIT HANDLER FOR ...;  -- handler H2
      *stmt1*;
      *stmt2*;
    END;
    ```

+   处理程序仅在声明它的块中有效，并且不能用于发生在该块外部的条件。在下面的示例中，处理程序`H1`仅在内部块中的*`stmt1`*中有效，而不适用于外部块中的*`stmt2`*：

    ```sql
    BEGIN -- outer block
      BEGIN -- inner block
        DECLARE EXIT HANDLER FOR ...;  -- handler H1
        *stmt1*;
      END;
      *stmt2*;
    END;
    ```

+   处理程序可以是特定的或一般的。特定处理程序是针对 MySQL 错误代码、`SQLSTATE`值或条件名称的。一般处理程序是针对`SQLWARNING`、`SQLEXCEPTION`或`NOT FOUND`类中的条件。条件特异性与条件优先级有关，如后面所述。

多个处理程序可以在不同的范围和具有不同的特异性中声明。例如，在外部块中可能有一个特定的 MySQL 错误代码处理程序，而在内部块中可能有一个一般的`SQLWARNING`处理程序。或者在同一块中可能有一个特定的 MySQL 错误代码处理程序和一般的`SQLWARNING`类处理程序。

处理程序是否被激活不仅取决于其自身的范围和条件值，还取决于其他处理程序的存在。当存储过程中发生条件时，服务器会在当前范围（当前`BEGIN ... END`块）中搜索适用的处理程序。如果没有适用的处理程序，则搜索会继续向外进行，直到找到每个连续包含范围（块）中的处理程序。当服务器在给定范围找到一个或多个适用的处理程序时，它会根据条件优先级在它们之间进行选择：

+   一个 MySQL 错误代码处理程序优先于一个`SQLSTATE`值处理程序。

+   一个`SQLSTATE`值处理程序优先于一般的`SQLWARNING`、`SQLEXCEPTION`或`NOT FOUND`处理程序。

+   一个`SQLEXCEPTION`处理程序优先于一个`SQLWARNING`处理程序。

+   可能存在几个具有相同优先级的适用处理程序。例如，一个语句可能生成多个具有不同错误代码的警告，对于每个警告都存在一个特定错误的处理程序。在这种情况下，服务器激活哪个处理程序的选择是不确定的，并且可能根据条件发生的情况而变化。

处理程序选择规则的一个含义是，如果不同作用域中存在多个适用的处理程序，则具有最局部作用域的处理程序优先于外部作用域中的处理程序，甚至优先于更具体条件的处理程序。

如果在条件发生时没有适当的处理程序，则采取的操作取决于条件的类别：

+   对于`SQLEXCEPTION`条件，存储程序在引发条件的语句处终止，就好像有一个`EXIT`处理程序。如果程序是由另一个存储程序调用的，则调用程序使用其自己的处理程序选择规则处理条件。

+   对于`SQLWARNING`条件，程序继续执行，就好像有一个`CONTINUE`处理程序。

+   对于`NOT FOUND`条件，如果条件是正常引发的，则操作是`CONTINUE`。如果是由`SIGNAL`或`RESIGNAL`引发的，则操作是`EXIT`。

以下示例演示了 MySQL 如何应用处理程序选择规则。

这个过程包含两个处理程序，一个用于特定的`SQLSTATE`值（`'42S02'`），用于尝试删除不存在表时发生的情况，另一个用于一般的`SQLEXCEPTION`类：

```sql
CREATE PROCEDURE p1()
BEGIN
  DECLARE CONTINUE HANDLER FOR SQLSTATE '42S02'
    SELECT 'SQLSTATE handler was activated' AS msg;
  DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    SELECT 'SQLEXCEPTION handler was activated' AS msg;

  DROP TABLE test.t;
END;
```

两个处理程序都在同一个块中声明并具有相同的作用域。然而，`SQLSTATE`处理程序优先于`SQLEXCEPTION`处理程序，因此如果表`t`不存在，则`DROP TABLE`语句引发一个激活`SQLSTATE`处理程序的条件：

```sql
mysql> CALL p1();
+--------------------------------+
| msg                            |
+--------------------------------+
| SQLSTATE handler was activated |
+--------------------------------+
```

这个过程包含相同的两个处理程序。但这次，`DROP TABLE`语句和`SQLEXCEPTION`处理程序在相对于`SQLSTATE`处理程序的内部块中：

```sql
CREATE PROCEDURE p2()
BEGIN -- outer block
    DECLARE CONTINUE HANDLER FOR SQLSTATE '42S02'
      SELECT 'SQLSTATE handler was activated' AS msg;
  BEGIN -- inner block
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
      SELECT 'SQLEXCEPTION handler was activated' AS msg;

    DROP TABLE test.t; -- occurs within inner block
  END;
END;
```

在这种情况下，更接近条件发生位置的处理程序优先。即使`SQLEXCEPTION`处理程序比`SQLSTATE`处理程序更一般，`SQLEXCEPTION`处理程序也会被激活：

```sql
mysql> CALL p2();
+------------------------------------+
| msg                                |
+------------------------------------+
| SQLEXCEPTION handler was activated |
+------------------------------------+
```

在这个过程中，处理程序之一在`DROP TABLE`语句的作用域内部声明：

```sql
CREATE PROCEDURE p3()
BEGIN -- outer block
  DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    SELECT 'SQLEXCEPTION handler was activated' AS msg;
  BEGIN -- inner block
    DECLARE CONTINUE HANDLER FOR SQLSTATE '42S02'
      SELECT 'SQLSTATE handler was activated' AS msg;
  END;

  DROP TABLE test.t; -- occurs within outer block
END;
```

只有`SQLEXCEPTION`处理程序适用，因为另一个处理程序不适用于`DROP TABLE`引发的条件：

```sql
mysql> CALL p3();
+------------------------------------+
| msg                                |
+------------------------------------+
| SQLEXCEPTION handler was activated |
+------------------------------------+
```

在这个过程中，两个处理程序都在`DROP TABLE`语句的作用域内部声明：

```sql
CREATE PROCEDURE p4()
BEGIN -- outer block
  BEGIN -- inner block
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
      SELECT 'SQLEXCEPTION handler was activated' AS msg;
    DECLARE CONTINUE HANDLER FOR SQLSTATE '42S02'
      SELECT 'SQLSTATE handler was activated' AS msg;
  END;

  DROP TABLE test.t; -- occurs within outer block
END;
```

由于它们不在`DROP TABLE`的范围内，因此都不适用。语句引发的条件未被处理，导致过程以错误终止：

```sql
mysql> CALL p4();
ERROR 1051 (42S02): Unknown table 'test.t'
```
