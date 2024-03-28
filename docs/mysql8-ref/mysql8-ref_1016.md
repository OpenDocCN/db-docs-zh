> 原文：[`dev.mysql.com/doc/refman/8.0/en/case.html`](https://dev.mysql.com/doc/refman/8.0/en/case.html)

#### 15.6.5.1 CASE Statement

```sql
CASE *case_value*
    WHEN *when_value* THEN *statement_list*
    [WHEN *when_value* THEN *statement_list*] ...
    [ELSE *statement_list*]
END CASE
```

或：

```sql
CASE
    WHEN *search_condition* THEN *statement_list*
    [WHEN *search_condition* THEN *statement_list*] ...
    [ELSE *statement_list*]
END CASE
```

存储程序的`CASE`语句实现了一个复杂的条件构造。

注意

还有一个与此处描述的`CASE` *语句*不同的`CASE` *运算符*。请参阅 Section 14.5, “Flow Control Functions”。`CASE`语句不能有`ELSE NULL`子句，并且以`END CASE`而不是`END`结束。

对于第一种语法，*`case_value`*是一个表达式。将此值与每个`WHEN`子句中的*`when_value`*表达式进行比较，直到找到相等的一个为止。找到相等的*`when_value`*后，执行相应的`THEN`子句*`statement_list`*。如果没有相等的*`when_value`*，则执行`ELSE`子句*`statement_list`*，如果有的话。

此语法不能用于与`NULL`进行相等性测试，因为`NULL = NULL`是错误的。请参阅 Section 5.3.4.6, “Working with NULL Values”。

对于第二种语法，每个`WHEN`子句*`search_condition`*表达式会被评估，直到其中一个为真，此时执行相应的`THEN`子句*`statement_list`*。如果没有相等的*`search_condition`*，则执行`ELSE`子句*`statement_list`*，如果有的话。

如果没有匹配被测试值的*`when_value`*或*`search_condition`*，并且`CASE`语句不包含`ELSE`子句，则会出现“CASE 语句未找到”错误。

每个*`statement_list`*由一个或多个 SQL 语句组成；不允许为空的*`statement_list`*。

为处理没有任何`WHEN`子句匹配的情况，请使用包含空的`BEGIN ... END`块的`ELSE`，如本示例所示。（此处`ELSE`子句中使用的缩进仅为了清晰起见，否则并不重要。）

```sql
DELIMITER |

CREATE PROCEDURE p()
  BEGIN
    DECLARE v INT DEFAULT 1;

    CASE v
      WHEN 2 THEN SELECT v;
      WHEN 3 THEN SELECT 0;
      ELSE
        BEGIN
        END;
    END CASE;
  END;
  |
```
