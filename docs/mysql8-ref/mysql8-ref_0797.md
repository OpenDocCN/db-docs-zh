# 14.5 流程控制函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html)

**表 14.7 流程控制运算符**

| 名称 | 描述 |
| --- | --- |
| `CASE` | Case 运算符 |
| `IF()` | 如果/否则构造 |
| `IFNULL()` | 如果/否则构造 |
| `NULLIF()` | 如果`expr1 = expr2`则返回`NULL` |

+   [`CASE *`value`* WHEN *`compare_value`* THEN *`result`* [WHEN *`compare_value`* THEN *`result`* ...] [ELSE *`result`*] END`](flow-control-functions.html#operator_case)

    [`CASE WHEN *`condition`* THEN *`result`* [WHEN *`condition`* THEN *`result`* ...] [ELSE *`result`*] END`](flow-control-functions.html#operator_case)

    第一个`CASE`语法返回第一个`*value`*=*`compare_value`*为真的`*result*`。第二个语法返回第一个为真的条件的结果。如果没有比较或条件为真，则返回`ELSE`后的结果，如果没有`ELSE`部分则返回`NULL`。

    注意

    此处描述的`CASE` *运算符*的语法与第 15.6.5.1 节“CASE 语句”中描述的 SQL `CASE` *语句*略有不同，用于存储程序内部。`CASE`语句不能有`ELSE NULL`子句，并以`END CASE`而不是`END`结束。

    `CASE`表达式的返回类型是所有结果值的聚合类型：

    +   如果所有类型都是数值型，则聚合类型也是数值型：

        +   如果至少有一个参数是双精度，则结果为双精度。

        +   否则，如果至少有一个参数是`DECIMAL`，则结果为`DECIMAL`。

        +   否则，结果是整数类型（有一个例外）：

            +   如果所有整数类型都是全部有符号或全部无符号，结果是相同符号且精度是所有指定整数类型中最高的（即`TINYINT`、`SMALLINT`、`MEDIUMINT`、`INT`或`BIGINT`）。

            +   如果有符号和无符号整数类型的组合，结果是有符号的，精度可能更高。例如，如果类型是有符号的`INT`和无符号的`INT`，结果是有符号的`BIGINT`。

            +   例外情况是无符号的`BIGINT`与任何有符号整数类型相结合。结果是具有足够精度和标度为 0 的`DECIMAL`。

    +   如果所有类型都是`BIT`，结果是`BIT`。否则，`BIT`参数被视为类似于`BIGINT`。

    +   如果所有类型都是`YEAR`，结果是`YEAR`。否则，`YEAR` 参数被视为类似于`INT`。

    +   如果所有类型都是字符字符串（`CHAR`或`VARCHAR`)，结果是具有由操作数的最长字符长度确定的最大长度的`VARCHAR`。

    +   如果所有类型都是字符或二进制字符串，结果是`VARBINARY`。

    +   `SET`和`ENUM`被视为类似于`VARCHAR`；结果是`VARCHAR`。

    +   如果所有类型都是`JSON`，结果是`JSON`。

    +   如果所有类型都是时间类型，结果就是时间类型：

        +   如果所有时间类型都是`DATE`、`TIME`或`TIMESTAMP`，结果分别是`DATE`、`TIME`或`TIMESTAMP`。

        +   否则，对于时间类型的混合，结果是`DATETIME`。

    +   如果所有类型都是`GEOMETRY`，结果就是`GEOMETRY`。

    +   如果任何类型是`BLOB`，结果是`BLOB`。

    +   对于所有其他类型组合，结果是`VARCHAR`。

    +   对于类型聚合，字面`NULL`操作数将被忽略。

    ```sql
    mysql> SELECT CASE 1 WHEN 1 THEN 'one'
     ->     WHEN 2 THEN 'two' ELSE 'more' END;
     -> 'one'
    mysql> SELECT CASE WHEN 1>0 THEN 'true' ELSE 'false' END;
     -> 'true'
    mysql> SELECT CASE BINARY 'B'
     ->     WHEN 'a' THEN 1 WHEN 'b' THEN 2 END;
     -> NULL
    ```

+   `IF(*`expr1`*,*`expr2`*,*`expr3`*)`

    如果*`expr1`*为`TRUE`（`*`expr1`* <> 0`且`*`expr1`* IS NOT NULL`），`IF()`返回*`expr2`*。否则，返回*`expr3`*。

    注意

    还有一个`IF` *语句*，与此处描述的`IF()` *函数*不同。请参见第 15.6.5.2 节，“IF 语句”。

    如果*`expr2`*或*`expr3`*中只有一个明确为`NULL`，则`IF()`函数的结果类型是非`NULL`表达式的类型。

    `IF()`的默认返回类型（当它存储到临时表时可能很重要）计算如下：

    +   如果*`expr2`*或*`expr3`*产生字符串，则结果是字符串。

        如果*`expr2`*和*`expr3`*都是字符串，则结果是区分大小写的，如果任一字符串是区分大小写的。

    +   如果*`expr2`*或*`expr3`*产生浮点值，则结果是浮点值。

    +   如果*`expr2`*或*`expr3`*产生整数，则结果是整数。

    ```sql
    mysql> SELECT IF(1>2,2,3);
     -> 3
    mysql> SELECT IF(1<2,'yes','no');
     -> 'yes'
    mysql> SELECT IF(STRCMP('test','test1'),'no','yes');
     -> 'no'
    ```

+   `IFNULL(*`expr1`*,*`expr2`*)`

    如果*`expr1`*不是`NULL`，`IFNULL()`返回*`expr1`*；否则返回*`expr2`*。

    ```sql
    mysql> SELECT IFNULL(1,0);
     -> 1
    mysql> SELECT IFNULL(NULL,10);
     -> 10
    mysql> SELECT IFNULL(1/0,10);
     -> 10
    mysql> SELECT IFNULL(1/0,'yes');
     -> 'yes'
    ```

    `IFNULL(*`expr1`*,*`expr2`*)`的默认返回类型是两个表达式中更“通用”的类型，按顺序为`STRING`、`REAL`或`INTEGER`。考虑基于表达式的表或 MySQL 必须在临时表中内部存储`IFNULL()`返回的值的情况：

    ```sql
    mysql> CREATE TABLE tmp SELECT IFNULL(1,'test') AS test;
    mysql> DESCRIBE tmp;
    +-------+--------------+------+-----+---------+-------+
    | Field | Type         | Null | Key | Default | Extra |
    +-------+--------------+------+-----+---------+-------+
    | test  | varbinary(4) | NO   |     |         |       |
    +-------+--------------+------+-----+---------+-------+
    ```

    在这个例子中，`test`列的类型是`VARBINARY(4)`(一个字符串类型)。

+   `NULLIF(*`expr1`*,*`expr2`*)`

    如果`*`expr1`* = *`expr2`*`为真，则返回`NULL`，否则返回*`expr1`*。这与`CASE WHEN *`expr1`* = *`expr2`* THEN NULL ELSE *`expr1`* END`相同。

    返回值与第一个参数具有相同的类型。

    ```sql
    mysql> SELECT NULLIF(1,1);
     -> NULL
    mysql> SELECT NULLIF(1,2);
     -> 1
    ```

    注意

    MySQL 在参数不相等时会对*`expr1`*进行两次评估。

MySQL 8.0.22 中这些函数对系统变量值的处理发生了变化。对于这些函数中的每一个，如果第一个参数仅包含在第二个参数使用的字符集和校对规则中存在的字符（且它是常量），则后者的字符集和校对规则用于进行比较。在 MySQL 8.0.22 及更高版本中，系统变量值被处理为具有相同字符集和校对规则的列值。一些使用这些函数与系统变量的查询可能会被拒绝，出现 Illegal mix of collations。在这种情况下，您应该将系统变量转换为正确的字符集和校对规则。
