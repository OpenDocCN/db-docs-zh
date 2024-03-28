> 原文：[`dev.mysql.com/doc/refman/8.0/en/nested-join-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/nested-join-optimization.html)

#### 10.2.1.8 嵌套连接优化

表达连接的语法允许嵌套连接。以下讨论涉及第 15.2.13.2 节，“JOIN 子句”中描述的连接语法。

与 SQL 标准相比，*`table_factor`*的语法有所扩展。后者仅接受*`table_reference`*，而不是在一对括号内列出它们。如果我们将*`table_reference`*项目列表中的每个逗号视为等同于内连接，则这是一种保守的扩展。例如：

```sql
SELECT * FROM t1 LEFT JOIN (t2, t3, t4)
                 ON (t2.a=t1.a AND t3.b=t1.b AND t4.c=t1.c)
```

等同于：

```sql
SELECT * FROM t1 LEFT JOIN (t2 CROSS JOIN t3 CROSS JOIN t4)
                 ON (t2.a=t1.a AND t3.b=t1.b AND t4.c=t1.c)
```

在 MySQL 中，`CROSS JOIN`在语法上等同于`INNER JOIN`；它们可以互换使用。在标准 SQL 中，它们不等效。`INNER JOIN`与`ON`子句一起使用；否则使用`CROSS JOIN`。

一般来说，可以忽略仅包含内连接操作的连接表达式中的括号。考虑这个连接表达式：

```sql
t1 LEFT JOIN (t2 LEFT JOIN t3 ON t2.b=t3.b OR t2.b IS NULL)
   ON t1.a=t2.a
```

删除括号并将操作分组到左侧后，该连接表达式转换为以下表达式：

```sql
(t1 LEFT JOIN t2 ON t1.a=t2.a) LEFT JOIN t3
    ON t2.b=t3.b OR t2.b IS NULL
```

然而，这两个表达式并不等效。为了看到这一点，假设表`t1`，`t2`和`t3`具有以下状态：

+   表`t1`包含行`(1)`，`(2)`

+   表`t2`包含行`(1,101)`

+   表`t3`包含行`(101)`

在这种情况下，第一个表达式返回包含行`(1,1,101,101)`，`(2,NULL,NULL,NULL)`的结果集，而第二个表达式返回行`(1,1,101,101)`，`(2,NULL,NULL,101)`：

```sql
mysql> SELECT *
       FROM t1
            LEFT JOIN
            (t2 LEFT JOIN t3 ON t2.b=t3.b OR t2.b IS NULL)
            ON t1.a=t2.a;
+------+------+------+------+
| a    | a    | b    | b    |
+------+------+------+------+
|    1 |    1 |  101 |  101 |
|    2 | NULL | NULL | NULL |
+------+------+------+------+

mysql> SELECT *
       FROM (t1 LEFT JOIN t2 ON t1.a=t2.a)
            LEFT JOIN t3
            ON t2.b=t3.b OR t2.b IS NULL;
+------+------+------+------+
| a    | a    | b    | b    |
+------+------+------+------+
|    1 |    1 |  101 |  101 |
|    2 | NULL | NULL |  101 |
+------+------+------+------+
```

在以下示例中，外连接操作与内连接操作一起使用：

```sql
t1 LEFT JOIN (t2, t3) ON t1.a=t2.a
```

该表达式无法转换为以下表达式：

```sql
t1 LEFT JOIN t2 ON t1.a=t2.a, t3
```

对于给定的表状态，这两个表达式返回不同的行集：

```sql
mysql> SELECT *
       FROM t1 LEFT JOIN (t2, t3) ON t1.a=t2.a;
+------+------+------+------+
| a    | a    | b    | b    |
+------+------+------+------+
|    1 |    1 |  101 |  101 |
|    2 | NULL | NULL | NULL |
+------+------+------+------+

mysql> SELECT *
       FROM t1 LEFT JOIN t2 ON t1.a=t2.a, t3;
+------+------+------+------+
| a    | a    | b    | b    |
+------+------+------+------+
|    1 |    1 |  101 |  101 |
|    2 | NULL | NULL |  101 |
+------+------+------+------+
```

因此，如果在具有外连接运算符的连接表达式中省略括号，可能会改变原始表达式的结果集。

更确切地说，在左外连接操作的右操作数和右连接操作的左操作数中不能忽略括号。换句话说，我们不能忽略外连接操作的内表达式的括号。其他操作数（外表的操作数）的括号可以忽略。

以下表达式：

```sql
(t1,t2) LEFT JOIN t3 ON P(t2.b,t3.b)
```

对于任何表`t1,t2,t3`和任何条件`P`，在属性`t2.b`和`t3.b`上：

```sql
t1, t2 LEFT JOIN t3 ON P(t2.b,t3.b)
```

每当连接表达式（*`joined_table`*）中连接操作的执行顺序不是从左到右时，我们谈论嵌套连接。考虑以下查询：

```sql
SELECT * FROM t1 LEFT JOIN (t2 LEFT JOIN t3 ON t2.b=t3.b) ON t1.a=t2.a
  WHERE t1.a > 1

SELECT * FROM t1 LEFT JOIN (t2, t3) ON t1.a=t2.a
  WHERE (t2.b=t3.b OR t2.b IS NULL) AND t1.a > 1
```

这些查询被认为包含这些嵌套连接：

```sql
t2 LEFT JOIN t3 ON t2.b=t3.b
t2, t3
```

在第一个查询中，嵌套连接是通过左连接操作形成的。在第二个查询中，它是通过内连接操作形成的。

在第一个查询中，括号可以省略：连接表达式的语法结构决定了连接操作的执行顺序相同。对于第二个查询，不能省略括号，尽管这里的连接表达式可以在没有括号的情况下明确解释。在我们的扩展语法中，第二个查询中`(t2, t3)`的括号是必需的，尽管理论上可以在没有它们的情况下解析查询：我们仍然会对查询有一个明确的语法结构，因为`LEFT JOIN`和`ON`扮演了表达式`(t2,t3)`的左右定界符的角色。

前面的例子演示了这些要点：

+   对于仅涉及内连接（而不是外连接）的连接表达式，可以去掉括号并从左到右评估连接。实际上，表可以以任何顺序评估。

+   一般来说，对于外连接或混合内连接的外连接，去掉括号可能会改变结果。

具有嵌套外连接的查询以与具有内连接相同的管道方式执行。更确切地说，利用了嵌套循环连接算法的一个变体。回想一下嵌套循环连接执行查询的算法（参见第 10.2.1.7 节，“嵌套循环连接算法”）。假设一个涉及 3 个表`T1,T2,T3`的连接查询具有以下形式：

```sql
SELECT * FROM T1 INNER JOIN T2 ON P1(T1,T2)
                 INNER JOIN T3 ON P2(T2,T3)
  WHERE P(T1,T2,T3)
```

这里，`P1(T1,T2)`和`P2(T3,T3)`是一些连接条件（基于表达式），而`P(T1,T2,T3)`是关于表`T1,T2,T3`列的条件。

嵌套循环连接算法将以以下方式执行此查询：

```sql
FOR each row t1 in T1 {
  FOR each row t2 in T2 such that P1(t1,t2) {
    FOR each row t3 in T3 such that P2(t2,t3) {
      IF P(t1,t2,t3) {
         t:=t1||t2||t3; OUTPUT t;
      }
    }
  }
}
```

表示通过连接行`t1`, `t2`和`t3`的列构造的行的符号`t1||t2||t3`。在以下一些示例中，表名出现的地方的`NULL`表示使用`NULL`作为该表的每列。例如，`t1||t2||NULL`表示通过连接行`t1`和`t2`的列，并为`t3`的每列使用`NULL`构造的行。这样的行被称为`NULL`-补充。

现在考虑一个具有嵌套外连接的查询：

```sql
SELECT * FROM T1 LEFT JOIN
              (T2 LEFT JOIN T3 ON P2(T2,T3))
              ON P1(T1,T2)
  WHERE P(T1,T2,T3)
```

对于这个查询，修改嵌套循环模式以获得：

```sql
FOR each row t1 in T1 {
  BOOL f1:=FALSE;
  FOR each row t2 in T2 such that P1(t1,t2) {
    BOOL f2:=FALSE;
    FOR each row t3 in T3 such that P2(t2,t3) {
      IF P(t1,t2,t3) {
        t:=t1||t2||t3; OUTPUT t;
      }
      f2=TRUE;
      f1=TRUE;
    }
    IF (!f2) {
      IF P(t1,t2,NULL) {
        t:=t1||t2||NULL; OUTPUT t;
      }
      f1=TRUE;
    }
  }
  IF (!f1) {
    IF P(t1,NULL,NULL) {
      t:=t1||NULL||NULL; OUTPUT t;
    }
  }
}
```

一般来说，在外连接操作的第一个内表的嵌套循环中，引入了一个标志，在循环之前关闭，在循环之后检查。当在外表的当前行中找到来自表示内操作数的表的匹配时，该标志被打开。如果在循环周期结束时标志仍然关闭，则没有找到外表当前行的匹配。在这种情况下，行将通过为内表的列补充`NULL`值。结果行将传递到输出的最终检查或下一个嵌套循环中，但仅当该行满足所有嵌套外连接的连接条件时。

在示例中，以下表达式表示的外连接表被嵌入：

```sql
(T2 LEFT JOIN T3 ON P2(T2,T3))
```

对于带有内连接的查询，优化器可以选择不同顺序的嵌套循环，例如这样一个顺序：

```sql
FOR each row t3 in T3 {
  FOR each row t2 in T2 such that P2(t2,t3) {
    FOR each row t1 in T1 such that P1(t1,t2) {
      IF P(t1,t2,t3) {
         t:=t1||t2||t3; OUTPUT t;
      }
    }
  }
}
```

对于带有外连接的查询，优化器只能选择外表的循环在内表的循环之前的顺序。因此，对于我们带有外连接的查询，只有一种嵌套顺序是可能的。对于以下查询，优化器评估了两种不同的嵌套。在这两种嵌套中，`T1`必须在外循环中处理，因为它用于外连接。`T2`和`T3`用于内连接，因此该连接必须在内循环中处理。然而，由于连接是内连接，`T2`和`T3`可以以任何顺序处理。

```sql
SELECT * T1 LEFT JOIN (T2,T3) ON P1(T1,T2) AND P2(T1,T3)
  WHERE P(T1,T2,T3)
```

一个嵌套评估`T2`，然后是`T3`：

```sql
FOR each row t1 in T1 {
  BOOL f1:=FALSE;
  FOR each row t2 in T2 such that P1(t1,t2) {
    FOR each row t3 in T3 such that P2(t1,t3) {
      IF P(t1,t2,t3) {
        t:=t1||t2||t3; OUTPUT t;
      }
      f1:=TRUE
    }
  }
  IF (!f1) {
    IF P(t1,NULL,NULL) {
      t:=t1||NULL||NULL; OUTPUT t;
    }
  }
}
```

另一个嵌套评估`T3`，然后是`T2`：

```sql
FOR each row t1 in T1 {
  BOOL f1:=FALSE;
  FOR each row t3 in T3 such that P2(t1,t3) {
    FOR each row t2 in T2 such that P1(t1,t2) {
      IF P(t1,t2,t3) {
        t:=t1||t2||t3; OUTPUT t;
      }
      f1:=TRUE
    }
  }
  IF (!f1) {
    IF P(t1,NULL,NULL) {
      t:=t1||NULL||NULL; OUTPUT t;
    }
  }
}
```

在讨论内连接的嵌套循环算法时，我们省略了一些细节，这些细节对查询执行性能的影响可能是巨大的。我们没有提到所谓的“推送下推”条件。假设我们的`WHERE`条件`P(T1,T2,T3)`可以用一个合取公式表示：

```sql
P(T1,T2,T2) = C1(T1) AND C2(T2) AND C3(T3).
```

在这种情况下，MySQL 实际上使用以下嵌套循环算法来执行带有内连接的查询：

```sql
FOR each row t1 in T1 such that C1(t1) {
  FOR each row t2 in T2 such that P1(t1,t2) AND C2(t2)  {
    FOR each row t3 in T3 such that P2(t2,t3) AND C3(t3) {
      IF P(t1,t2,t3) {
         t:=t1||t2||t3; OUTPUT t;
      }
    }
  }
}
```

您可以看到每个连接`C1(T1)`，`C2(T2)`，`C3(T3)`都被推出最内部循环到最外部循环，其中可以进行评估。如果`C1(T1)`是一个非常严格的条件，这种条件下推可能会大大减少传递给内部循环的表`T1`的行数。结果，查询的执行时间可能会极大地改善。

对于带有外连接的查询，`WHERE`条件只有在发现当前外表行在内表中有匹配时才会被检查。因此，将条件推出内部嵌套循环的优化不能直接应用于带有外连接的查询。在这里，我们必须引入由标志保护的条件推送下推谓词，当遇到匹配时这些标志被打开。

回想一下带有外连接的这个例子：

```sql
P(T1,T2,T3)=C1(T1) AND C(T2) AND C3(T3)
```

对于那个例子，使用受保护的推送下推条件的嵌套循环算法如下：

```sql
FOR each row t1 in T1 such that C1(t1) {
  BOOL f1:=FALSE;
  FOR each row t2 in T2
      such that P1(t1,t2) AND (f1?C2(t2):TRUE) {
    BOOL f2:=FALSE;
    FOR each row t3 in T3
        such that P2(t2,t3) AND (f1&&f2?C3(t3):TRUE) {
      IF (f1&&f2?TRUE:(C2(t2) AND C3(t3))) {
        t:=t1||t2||t3; OUTPUT t;
      }
      f2=TRUE;
      f1=TRUE;
    }
    IF (!f2) {
      IF (f1?TRUE:C2(t2) && P(t1,t2,NULL)) {
        t:=t1||t2||NULL; OUTPUT t;
      }
      f1=TRUE;
    }
  }
  IF (!f1 && P(t1,NULL,NULL)) {
      t:=t1||NULL||NULL; OUTPUT t;
  }
}
```

一般来说，推送下推谓词可以从连接条件中提取，例如`P1(T1,T2)`和`P(T2,T3)`。在这种情况下，推送下推谓词也受到一个标志的保护，该标志防止对由相应外连接操作生成的`NULL`-补充行进行谓词检查。

在相同的嵌套连接中，如果由`WHERE`条件引起，从一个内表到另一个内表的键访问是被禁止的。
