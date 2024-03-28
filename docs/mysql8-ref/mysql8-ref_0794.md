# 14.4.2 比较函数和运算符

> 原文：[`dev.mysql.com/doc/refman/8.0/en/comparison-operators.html`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html)

**表 14.4 比较运算符**

| 名称 | 描述 |
| --- | --- |
| `>` | 大于运算符 |
| `>=` | 大于或等于运算符 |
| `<` | 小于运算符 |
| `<>`, `!=` | 不等运算符 |
| `<=` | 小于或等于运算符 |
| `<=>` | NULL 安全等于运算符 |
| `=` | 等于运算符 |
| `BETWEEN ... AND ...` | 值是否在一系列值范围内 |
| `COALESCE()` | 返回第一个非 NULL 参数 |
| `GREATEST()` | 返回最大的参数 |
| `IN()` | 值是否在一组值内 |
| `INTERVAL()` | 返回小于第一个参数的参数的索引 |
| `IS` | 测试值是否为布尔值 |
| `IS NOT` | 测试值是否为布尔值 |
| `IS NOT NULL` | 非 NULL 值测试 |
| `IS NULL` | NULL 值测试 |
| `ISNULL()` | 测试参数是否为 NULL |
| `LEAST()` | 返回最小的参数 |
| `LIKE` | 简单的模式匹配 |
| `NOT BETWEEN ... AND ...` | 值是否不在一系列值范围内 |
| `NOT IN()` | 值是否不在一组值内 |
| `NOT LIKE` | 简单模式匹配的否定 |
| `STRCMP()` | 比较两个字符串 |
| 名称 | 描述 |

比较操作的结果为`1`（`TRUE`）、`0`（`FALSE`）或`NULL`。这些操作适用于数字和字符串。字符串会根据需要自动转换为数字，数字也会转换为字符串。

下列关系比较运算符可用于比较标量操作数以及行操作数：

```sql
=  >  <  >=  <=  <>  !=
```

本节后面对这些运算符的描述详细说明了它们如何与行操作数一起工作。有关在行子查询上下文中的行比较的其他示例，请参见第 15.2.15.5 节，“行子查询”。

本节中的一些函数返回除`1`（`TRUE`）、`0`（`FALSE`）或`NULL`之外的值。`LEAST()`和`GREATEST()`就是这样的函数的例子；第 14.3 节，“表达式评估中的类型转换”描述了这些函数执行比较操作以确定它们的返回值的规则。

注意

在 MySQL 的早期版本中，当评估包含`LEAST()`或`GREATEST()`的表达式时，服务器尝试猜测函数的使用上下文，并将函数的参数强制转换为整个表达式的数据类型。例如，对于`LEAST("11", "45", "2")`的参数将作为字符串进行评估和排序，因此该表达式返回`"11"`。在 MySQL 8.0.3 及更早版本中，当评估表达式`LEAST("11", "45", "2") + 0`时，服务器在对其进行排序之前将参数转换为整数（预期将整数 0 添加到结果），从而返回 2。

从 MySQL 8.0.4 开始，服务器不再尝试以这种方式推断上下文。相反，函数将使用提供的参数执行，仅在它们不全为相同类型时对一个或多个参数执行数据类型转换。现在，任何使用返回值的表达式强制执行的类型强制转换都是在函数执行后执行的。这意味着，在 MySQL 8.0.4 及更高版本中，`LEAST("11", "45", "2") + 0`计算为`"11" + 0`，因此为整数 11。 (Bug #83895, Bug #25123839)

要将值转换为特定类型以进行比较，可以使用`CAST()`函数。字符串值可以使用`CONVERT()`将其转换为不同的字符集。参见第 14.10 节，“转换函数和运算符”。

默认情况下，字符串比较不区分大小写，并使用当前字符集。默认为`utf8mb4`。

+   `=`

    等于：

    ```sql
    mysql> SELECT 1 = 0;
     -> 0
    mysql> SELECT '0' = 0;
     -> 1
    mysql> SELECT '0.0' = 0;
     -> 1
    mysql> SELECT '0.01' = 0;
     -> 0
    mysql> SELECT '.01' = 0.01;
     -> 1
    ```

    对于行比较，`(a, b) = (x, y)`等同于：

    ```sql
    (a = x) AND (b = y)
    ```

+   `<=>`

    `NULL`-安全等于。此运算符执行类似于`=`运算符的相等比较，但如果两个操作数都为`NULL`，则返回`1`而不是`NULL`，如果一个操作数为`NULL`，则返回`0`而不是`NULL`。

    `<=>`运算符等同于标准 SQL 的`IS NOT DISTINCT FROM`运算符。

    ```sql
    mysql> SELECT 1 <=> 1, NULL <=> NULL, 1 <=> NULL;
     -> 1, 1, 0
    mysql> SELECT 1 = 1, NULL = NULL, 1 = NULL;
     -> 1, NULL, NULL
    ```

    对于行比较，`(a, b) <=> (x, y)` 等同于：

    ```sql
    (a <=> x) AND (b <=> y)
    ```

+   `<>`, `!=`

    不等于：

    ```sql
    mysql> SELECT '.01' <> '0.01';
     -> 1
    mysql> SELECT .01 <> '0.01';
     -> 0
    mysql> SELECT 'zapp' <> 'zappp';
     -> 1
    ```

    对于行比较，`(a, b) <> (x, y)` 和 `(a, b) != (x, y)` 等同于：

    ```sql
    (a <> x) OR (b <> y)
    ```

+   `<=`

    小于或等于：

    ```sql
    mysql> SELECT 0.1 <= 2;
     -> 1
    ```

    对于行比较，`(a, b) <= (x, y)` 等同于：

    ```sql
    (a < x) OR ((a = x) AND (b <= y))
    ```

+   `<`

    小于：

    ```sql
    mysql> SELECT 2 < 2;
     -> 0
    ```

    对于行比较，`(a, b) < (x, y)` 等同于：

    ```sql
    (a < x) OR ((a = x) AND (b < y))
    ```

+   `>=`

    大于或等于：

    ```sql
    mysql> SELECT 2 >= 2;
     -> 1
    ```

    对于行比较，`(a, b) >= (x, y)` 等同于：

    ```sql
    (a > x) OR ((a = x) AND (b >= y))
    ```

+   `>`

    大于：

    ```sql
    mysql> SELECT 2 > 2;
     -> 0
    ```

    对于行比较，`(a, b) > (x, y)` 等同于：

    ```sql
    (a > x) OR ((a = x) AND (b > y))
    ```

+   `*`expr`* BETWEEN *`min`* AND *`max`*`

    如果*`expr`*大于或等于*`min`*且*`expr`*小于或等于*`max`*，`BETWEEN`返回`1`，否则返回`0`。如果所有参数类型相同，则这等同于表达式`(*`min`* <= *`expr`* AND *`expr`* <= *`max`*)`。否则，根据第 14.3 节，“表达式求值中的类型转换”中描述的规则进行类型转换，但应用于所有三个参数。

    ```sql
    mysql> SELECT 2 BETWEEN 1 AND 3, 2 BETWEEN 3 and 1;
     -> 1, 0
    mysql> SELECT 1 BETWEEN 2 AND 3;
     -> 0
    mysql> SELECT 'b' BETWEEN 'a' AND 'c';
     -> 1
    mysql> SELECT 2 BETWEEN 2 AND '3';
     -> 1
    mysql> SELECT 2 BETWEEN 2 AND 'x-3';
     -> 0
    ```

    对于使用`BETWEEN`与日期或时间值时，最佳结果是使用`CAST()`显式将值转换为所需的数据类型。例如：如果要比较一个`DATETIME`与两个`DATE`值，将`DATE`值转换为`DATETIME`值。如果在与`DATE`比较中使用字符串常量如`'2001-1-1'`，则将字符串转换为`DATE`。

+   `*`expr`* NOT BETWEEN *`min`* AND *`max`*`

    这与`NOT (*`expr`* BETWEEN *`min`* AND *`max`*)`相同。

+   `COALESCE(*`value`*,...)`

    返回列表中第一个非`NULL`值，如果没有非`NULL`值则返回`NULL`。

    `COALESCE()`的返回类型是参数类型的聚合类型。

    ```sql
    mysql> SELECT COALESCE(NULL,1);
     -> 1
    mysql> SELECT COALESCE(NULL,NULL,NULL);
     -> NULL
    ```

+   `GREATEST(*`value1`*,*`value2`*,...)`

    有两个或更多参数时，返回最大值的参数。参数使用与`LEAST()`相同的规则进行比较。

    ```sql
    mysql> SELECT GREATEST(2,0);
     -> 2
    mysql> SELECT GREATEST(34.0,3.0,5.0,767.0);
     -> 767.0
    mysql> SELECT GREATEST('B','A','C');
     -> 'C'
    ```

    `GREATEST()`如果任何参数为`NULL`，则返回`NULL`。

+   `*`expr`* IN (*`value`*,...)`

    如果*`expr`*等于`IN()`列表中的任何一个值，则返回`1`（true），否则返回`0`（false）。

    根据第 14.3 节，“表达式评估中的类型转换”中描述的规则进行类型转换，应用于所有参数。如果`IN()`列表中的值不需要类型转换，它们都是相同类型的非`JSON`常量，并且*`expr`*可以与它们中的每一个作为相同类型的值进行比较（可能经过类型转换），则会进行优化。列表中的值被排序，使用二分查找来搜索*`expr`*，使得`IN()`操作非常快速。

    ```sql
    mysql> SELECT 2 IN (0,3,5,7);
     -> 0
    mysql> SELECT 'wefwf' IN ('wee','wefwf','weg');
     -> 1
    ```

    `IN()`可以用于比较行构造：

    ```sql
    mysql> SELECT (3,4) IN ((1,2), (3,4));
     -> 1
    mysql> SELECT (3,4) IN ((1,2), (3,5));
     -> 0
    ```

    永远不要在`IN()`列表中混合引号和非引号值，因为引号值（如字符串）和非引号值（如数字）的比较规则不同。因此，混合类型可能导致不一致的结果。例如，不要像这样编写`IN()`表达式：

    ```sql
    SELECT val1 FROM tbl1 WHERE val1 IN (1,2,'a');
    ```

    相反，应该这样写：

    ```sql
    SELECT val1 FROM tbl1 WHERE val1 IN ('1','2','a');
    ```

    隐式类型转换可能会产生令人费解的结果：

    ```sql
    mysql> SELECT 'a' IN (0), 0 IN ('b');
     -> 1, 1
    ```

    在这两种情况下，比较值被转换为浮点值，每种情况下均产生 0.0，并且比较结果为 1（true）。

    `IN()`列表中的值的数量仅受`max_allowed_packet`值的限制。

    为了符合 SQL 标准，`IN()`不仅在左侧表达式为`NULL`时返回`NULL`，而且在列表中找不到匹配项且列表中的一个表达式为`NULL`时也返回`NULL`。

    `IN()`语法也可以用于编写某些类型的子查询。请参见第 15.2.15.3 节，“带有 ANY、IN 或 SOME 的子查询”。

+   `*`expr`* NOT IN (*`value`*,...)`

    这与`NOT (*`expr`* IN (*`value`*,...))`相同。

+   `INTERVAL(*`N`*,*`N1`*,*`N2`*,*`N3`*,...)`

    如果*`N`* ≤ *`N1`*，则返回`0`，如果*`N`* ≤ *`N2`*等等，或者如果*`N`*为`NULL`，则返回`-1`。所有参数都被视为整数。对于这个函数能够正确工作，需要满足*`N1`* ≤ *`N2`* ≤ *`N3`* ≤ `...` ≤ *`Nn`*。这是因为使用了二分查找（非常快速）。

    ```sql
    mysql> SELECT INTERVAL(23, 1, 15, 17, 30, 44, 200);
     -> 3
    mysql> SELECT INTERVAL(10, 1, 10, 100, 1000);
     -> 2
    mysql> SELECT INTERVAL(22, 23, 30, 44, 200);
     -> 0
    ```

+   `IS *`boolean_value`*`

    测试一个值是否等于布尔值，其中*`boolean_value`*可以是`TRUE`、`FALSE`或`UNKNOWN`。

    ```sql
    mysql> SELECT 1 IS TRUE, 0 IS FALSE, NULL IS UNKNOWN;
     -> 1, 1, 1
    ```

+   `IS NOT *`boolean_value`*`

    测试一个值是否等于布尔值，其中*`boolean_value`*可以是`TRUE`、`FALSE`或`UNKNOWN`。

    ```sql
    mysql> SELECT 1 IS NOT UNKNOWN, 0 IS NOT UNKNOWN, NULL IS NOT UNKNOWN;
     -> 1, 1, 0
    ```

+   `IS NULL`

    测试一个值是否为`NULL`。

    ```sql
    mysql> SELECT 1 IS NULL, 0 IS NULL, NULL IS NULL;
     -> 0, 0, 1
    ```

    为了与 ODBC 程序良好配合，MySQL 在使用`IS NULL`时支持以下额外功能：

    +   如果`sql_auto_is_null`变量设置为 1，则在成功插入自动生成的`AUTO_INCREMENT`值的语句之后，可以通过发出以下形式的语句找到该值：

        ```sql
        SELECT * FROM *tbl_name* WHERE *auto_col* IS NULL
        ```

        如果语句返回一行，则返回的值与调用`LAST_INSERT_ID()`函数的结果相同。有关详细信息，包括多行插入后的返回值，请参见第 14.15 节，“信息函数”。如果没有成功插入`AUTO_INCREMENT`值，则`SELECT`语句不返回任何行。

        通过设置`sql_auto_is_null = 0`可以禁用使用`IS NULL`比较来检索`AUTO_INCREMENT`值的行为。请参见第 7.1.8 节，“服务器系统变量”。

        `sql_auto_is_null`的默认值为 0。

    +   对于声明为`NOT NULL`的`DATE`和`DATETIME`列，可以通过类似以下语句找到特殊日期`'0000-00-00'`：

        ```sql
        SELECT * FROM *tbl_name* WHERE *date_column* IS NULL
        ```

        这是为了使一些 ODBC 应用程序正常工作而需要的，因为 ODBC 不支持`'0000-00-00'`日期值。

        参见获取自增值，以及 Connector/ODBC 连接参数中`FLAG_AUTO_IS_NULL`选项的描述。

+   `IS NOT NULL`

    测试一个值是否不为`NULL`。

    ```sql
    mysql> SELECT 1 IS NOT NULL, 0 IS NOT NULL, NULL IS NOT NULL;
     -> 1, 1, 0
    ```

+   `ISNULL(*`expr`*)`

    如果*`expr`*为`NULL`，`ISNULL()`返回`1`，否则返回`0`。

    ```sql
    mysql> SELECT ISNULL(1+1);
     -> 0
    mysql> SELECT ISNULL(1/0);
     -> 1
    ```

    `ISNULL()`可用于代替`=`来测试一个值是否为`NULL`。（使用`=`将值与`NULL`进行比较总是返回`NULL`。）

    `ISNULL()`函数与`IS NULL`比较运算符共享一些特殊行为。请参见`IS NULL`的描述。

+   `LEAST(*`value1`*,*`value2`*,...)`

    对于两个或更多参数，返回最小值的参数。参数将根据以下规则进行比较：

    +   如果任何参数为`NULL`，则结果为`NULL`。不需要进行比较。

    +   如果所有参数都是整数值，则它们将作为整数进行比较。

    +   如果至少一个参数是双精度，则它们将作为双精度值进行比较。否则，如果至少一个参数是`DECIMAL` - DECIMAL, NUMERIC")值，则它们将作为`DECIMAL` - DECIMAL, NUMERIC")值进行比较。

    +   如果参数包含数字和字符串的混合，则它们将作为字符串进行比较。

    +   如果任何参数是非二进制（字符）字符串，则参数将作为非二进制字符串进行比较。

    +   在所有其他情况下，参数将作为二进制字符串进行比较。

    `LEAST()`的返回类型是比较参数类型的聚合类型。

    ```sql
    mysql> SELECT LEAST(2,0);
     -> 0
    mysql> SELECT LEAST(34.0,3.0,5.0,767.0);
     -> 3.0
    mysql> SELECT LEAST('B','A','C');
     -> 'A'
    ```
