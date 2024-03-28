# 14.4.4 分配运算符

> 原文：[`dev.mysql.com/doc/refman/8.0/en/assignment-operators.html`](https://dev.mysql.com/doc/refman/8.0/en/assignment-operators.html)

**表 14.6 分配运算符**

| 名称 | 描述 |
| --- | --- |
| `:=` | 分配一个值 |
| `=` | 分配一个值（作为`SET`语句的一部分，或作为`UPDATE`语句中的`SET`子句） |

+   `:=`

    分配运算符。导致运算符左侧的用户变量取右侧的值。右侧的值可以是文字值，存储值的另一个变量，或产生标量值的任何合法表达式，包括查询的结果（前提是这个值是标量值）。您可以在同一条`SET`语句中执行多个分配。您可以在同一条语句中执行多个分配。

    与`=`不同，`:=`运算符永远不会被解释为比较运算符。这意味着您可以在任何有效的 SQL 语句（不仅仅是在`SET`语句中）中使用`:=`来为变量分配一个值。

    ```sql
    mysql> SELECT @var1, @var2;
     -> NULL, NULL
    mysql> SELECT @var1 := 1, @var2;
     -> 1, NULL
    mysql> SELECT @var1, @var2;
     -> 1, NULL
    mysql> SELECT @var1, @var2 := @var1;
     -> 1, 1
    mysql> SELECT @var1, @var2;
     -> 1, 1

    mysql> SELECT @var1:=COUNT(*) FROM t1;
     -> 4
    mysql> SELECT @var1;
     -> 4
    ```

    除了`SELECT`之外，您还可以在其他语句中使用`:=`进行值分配，例如`UPDATE`，如下所示：

    ```sql
    mysql> SELECT @var1;
     -> 4
    mysql> SELECT * FROM t1;
     -> 1, 3, 5, 7

    mysql> UPDATE t1 SET c1 = 2 WHERE c1 = @var1:= 1;
    Query OK, 1 row affected (0.00 sec)
    Rows matched: 1  Changed: 1  Warnings: 0

    mysql> SELECT @var1;
     -> 1
    mysql> SELECT * FROM t1;
     -> 2, 3, 5, 7
    ```

    虽然在单个 SQL 语句中使用`:=`运算符既可以设置变量的值又可以读取变量的值，但不建议这样做。第 11.4 节，“用户定义变量”解释了为什么应该避免这样做。

+   `=`

    此运算符用于在下面两种情况下执行值分配，分别在接下来的两段中描述。

    在`SET`语句中，`=`被视为一个赋值运算符，导致操作符左侧的用户变量取得右侧的值。（换句话说，在`SET`语句中使用时，`=`与`:=`的作用是相同的。）右侧的值可以是一个字面值，存储值的另一个变量，或者产生标量值的任何合法表达式，包括查询的结果（前提是这个值是一个标量值）。你可以在同一个`SET`语句中执行多个赋值操作。

    在`UPDATE`语句的`SET`子句中，`=`也充当一个赋值运算符；然而，在这种情况下，它会导致操作符左侧命名的列取得右侧给定的值，前提是`UPDATE`中的任何`WHERE`条件都得到满足。你可以在同一个`SET`子句的`UPDATE`语句中进行多个赋值操作。

    在任何其他上下文中，`=`被视为一个比较运算符。

    ```sql
    mysql> SELECT @var1, @var2;
     -> NULL, NULL
    mysql> SELECT @var1 := 1, @var2;
     -> 1, NULL
    mysql> SELECT @var1, @var2;
     -> 1, NULL
    mysql> SELECT @var1, @var2 := @var1;
     -> 1, 1
    mysql> SELECT @var1, @var2;
     -> 1, 1
    ```

    欲了解更多信息，请参阅第 15.7.6.1 节，“变量赋值的 SET 语法”，第 15.2.17 节，“UPDATE 语句”，以及第 15.2.15 节，“子查询”。
