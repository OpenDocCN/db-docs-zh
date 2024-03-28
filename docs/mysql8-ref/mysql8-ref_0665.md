# 11.5 表达式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/expressions.html`](https://dev.mysql.com/doc/refman/8.0/en/expressions.html)

此部分列出了在 MySQL 中表达式必须遵循的语法规则，并提供了关于可能出现在表达式中的术语类型的附加信息。

+   表达式语法

+   表达式术语注释

+   时间间隔

### 表达式语法

以下语法规则定义了 MySQL 中的表达式语法。这里显示的语法基于 MySQL 源分发中 `sql/sql_yacc.yy` 文件中给出的语法。有关一些表达式术语的附加信息，请参见表达式术语注释。

```sql
*expr*:
    *expr* OR *expr*
  | *expr* || *expr*
  | *expr* XOR *expr*
  | *expr* AND *expr*
  | *expr* && *expr*
  | NOT *expr*
  | ! *expr*
  | *boolean_primary* IS [NOT] {TRUE | FALSE | UNKNOWN}
  | *boolean_primary*

*boolean_primary*:
    *boolean_primary* IS [NOT] NULL
  | *boolean_primary* <=> *predicate*
  | *boolean_primary* *comparison_operator* *predicate*
  | *boolean_primary* *comparison_operator* {ALL | ANY} (*subquery*)
  | *predicate*

*comparison_operator*: = | >= | > | <= | < | <> | !=

*predicate*:
    *bit_expr* [NOT] IN (*subquery*)
  | *bit_expr* [NOT] IN (*expr* [, *expr*] ...)
  | *bit_expr* [NOT] BETWEEN *bit_expr* AND *predicate*
  | *bit_expr* SOUNDS LIKE *bit_expr*
  | *bit_expr* [NOT] LIKE *simple_expr* [ESCAPE *simple_expr*]
  | *bit_expr* [NOT] REGEXP *bit_expr*
  | *bit_expr*

*bit_expr*:
    *bit_expr* | *bit_expr*
  | *bit_expr* & *bit_expr*
  | *bit_expr* << *bit_expr*
  | *bit_expr* >> *bit_expr*
  | *bit_expr* + *bit_expr*
  | *bit_expr* - *bit_expr*
  | *bit_expr* * *bit_expr*
  | *bit_expr* / *bit_expr*
  | *bit_expr* DIV *bit_expr*
  | *bit_expr* MOD *bit_expr*
  | *bit_expr* % *bit_expr*
  | *bit_expr* ^ *bit_expr*
  | *bit_expr* + *interval_expr*
  | *bit_expr* - *interval_expr*
  | *simple_expr*

*simple_expr*:
    *literal*
  | *identifier*
  | *function_call*
  | *simple_expr* COLLATE *collation_name*
  | *param_marker*
  | *variable*
  | *simple_expr* || *simple_expr*
  | + *simple_expr*
  | - *simple_expr*
  | ~ *simple_expr*
  | ! *simple_expr*
  | BINARY *simple_expr*
  | (*expr* [, *expr*] ...)
  | ROW (*expr*, *expr* [, *expr*] ...)
  | (*subquery*)
  | EXISTS (*subquery*)
  | {*identifier* *expr*}
  | *match_expr*
  | *case_expr*
  | *interval_expr*
```

关于运算符优先级，请参见第 14.4.1 节，“运算符优先级”。某些运算符的优先级和含义取决于 SQL 模式：

+   默认情况下，`||` 是逻辑 `OR` 运算符。启用 `PIPES_AS_CONCAT` 后，`||` 是字符串连接运算符，优先级介于 `^` 和一元运算符之间。

+   默认情况下，`!` 的优先级高于 `NOT`。启用 `HIGH_NOT_PRECEDENCE` 后，`!` 和 `NOT` 具有相同的优先级。

请参见第 7.1.11 节，“服务器 SQL 模式”。

### 表达式术语注释

有关文字值语法，请参见第 11.1 节，“文字值”。

有关标识符语法，请参见第 11.2 节，“模式对象名称”。

变量可以是用户变量、系统变量、存储过程本地变量或参数：

+   用户变量：第 11.4 节，“用户定义变量”

+   系统变量：第 7.1.9 节，“使用系统变量”

+   存储过程本地变量：第 15.6.4.1 节，“本地变量 DECLARE 语句”

+   存储过程参数：第 15.1.17 节，“CREATE PROCEDURE 和 CREATE FUNCTION 语句”

*`param_marker`* 是用于占位符的预处理语句中使用的 `?`。请参见第 15.5.1 节，“PREPARE 语句”。

`(*`subquery`*)` 表示返回单个值的子查询；即标量子查询。参见 Section 15.2.15.1, “标量操作数的子查询”。

`{*`identifier`* *`expr`*}` 是 ODBC 转义语法，为了兼容 ODBC 而被接受。值为 *`expr`*。语法中的 `{` 和 `}` 大括号应该按照字面意义书写；它们不是在其他语法描述中使用的元语法。

*`match_expr`* 表示一个 `MATCH` 表达式。参见 Section 14.9, “全文搜索函数”。

*`case_expr`* 表示一个 `CASE` 表达式。参见 Section 14.5, “流程控制函数”。

*`interval_expr`* 表示一个时间间隔。参见 时间间隔。

### 时间间隔

*`interval_expr`* 在表达式中表示一个时间间隔。间隔具有以下语法：

```sql
INTERVAL *expr* *unit*
```

*`expr`* 代表一个数量。*`unit`* 代表用于解释数量的单位；它是一个类似于 `小时`, `天` 或 `周` 的指定符号。`INTERVAL` 关键字和 *`unit`* 指定符号不区分大小写。

以下表格显示了每个 *`unit`* 值的 *`expr`* 参数的预期形式。

**表 11.2 时间间隔表达式和单位参数**

| *`unit`* 值 | 预期的 *`expr`* 格式 |
| --- | --- |
| `MICROSECOND` | `微秒` |
| `SECOND` | `秒` |
| `MINUTE` | `分钟` |
| `HOUR` | `小时` |
| `DAY` | `天` |
| `WEEK` | `周` |
| `MONTH` | `月` |
| `QUARTER` | `季度` |
| `YEAR` | `年` |
| `SECOND_MICROSECOND` | `'秒.微秒'` |
| `MINUTE_MICROSECOND` | `'分钟:秒.微秒'` |
| `MINUTE_SECOND` | `'分钟:秒'` |
| `HOUR_MICROSECOND` | `'小时:分钟:秒.微秒'` |
| `HOUR_SECOND` | `'小时:分钟:秒'` |
| `HOUR_MINUTE` | `'小时:分钟'` |
| `DAY_MICROSECOND` | `'天 小时:分钟:秒.微秒'` |
| `DAY_SECOND` | `'天 小时:分钟:秒'` |
| `DAY_MINUTE` | `'天 小时:分钟'` |
| `DAY_HOUR` | `'天 小时'` |
| `YEAR_MONTH` | `'年-月'` |
| *`unit`* 值 | 预期的 *`expr`* 格式 |

MySQL 允许在 *`expr`* 格式中使用任何标点符号分隔符。表中显示的是建议的分隔符。

时间间隔用于某些函数，例如 `DATE_ADD()` 和 `DATE_SUB()`： 

```sql
mysql> SELECT DATE_ADD('2018-05-01',INTERVAL 1 DAY);
 -> '2018-05-02'
mysql> SELECT DATE_SUB('2018-05-01',INTERVAL 1 YEAR);
 -> '2017-05-01'
mysql> SELECT DATE_ADD('2020-12-31 23:59:59',
 ->                 INTERVAL 1 SECOND);
 -> '2021-01-01 00:00:00'
mysql> SELECT DATE_ADD('2018-12-31 23:59:59',
 ->                 INTERVAL 1 DAY);
 -> '2019-01-01 23:59:59'
mysql> SELECT DATE_ADD('2100-12-31 23:59:59',
 ->                 INTERVAL '1:1' MINUTE_SECOND);
 -> '2101-01-01 00:01:00'
mysql> SELECT DATE_SUB('2025-01-01 00:00:00',
 ->                 INTERVAL '1 1:1:1' DAY_SECOND);
 -> '2024-12-30 22:58:59'
mysql> SELECT DATE_ADD('1900-01-01 00:00:00',
 ->                 INTERVAL '-1 10' DAY_HOUR);
 -> '1899-12-30 14:00:00'
mysql> SELECT DATE_SUB('1998-01-02', INTERVAL 31 DAY);
 -> '1997-12-02'
mysql> SELECT DATE_ADD('1992-12-31 23:59:59.000002',
 ->            INTERVAL '1.999999' SECOND_MICROSECOND);
 -> '1993-01-01 00:00:01.000001'
```

也可以使用 `INTERVAL` 与 `+` 或 `-` 运算符在表达式中执行时间算术：

```sql
date + INTERVAL *expr* *unit*
date - INTERVAL *expr* *unit*
```

如果另一侧的表达式是日期或日期时间值，则`+`运算符的两侧都允许`INTERVAL *`expr`* *`unit`*`。对于`-`运算符，只允许在右侧使用`INTERVAL *`expr`* *`unit`*，因为从间隔中减去日期或日期时间值是没有意义的。

```sql
mysql> SELECT '2018-12-31 23:59:59' + INTERVAL 1 SECOND;
 -> '2019-01-01 00:00:00'
mysql> SELECT INTERVAL 1 DAY + '2018-12-31';
 -> '2019-01-01'
mysql> SELECT '2025-01-01' - INTERVAL 1 SECOND;
 -> '2024-12-31 23:59:59'
```

`EXTRACT()`函数使用与`DATE_ADD()`或`DATE_SUB()`相同类型的*`unit`*指示符，但是从日期中提取部分而不是执行日期算术：

```sql
mysql> SELECT EXTRACT(YEAR FROM '2019-07-02');
 -> 2019
mysql> SELECT EXTRACT(YEAR_MONTH FROM '2019-07-02 01:02:03');
 -> 201907
```

时间间隔可以在`CREATE EVENT`语句中使用：

```sql
CREATE EVENT myevent
    ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 HOUR
    DO
      UPDATE myschema.mytable SET mycol = mycol + 1;
```

如果您指定的间隔值太短（不包括从*`unit`*关键字中预期的所有间隔部分），MySQL 会假定您省略了间隔值的最左边部分。例如，如果您指定了`DAY_SECOND`作为*`unit`*，则*`expr`*的值应包含天、小时、分钟和秒部分。如果您指定类似 `'1:10'` 的值，MySQL 会假定缺少天数和小时数部分，该值表示分钟和秒数。换句话说，`'1:10' DAY_SECOND`被解释为等同于`'1:10' MINUTE_SECOND`。这类似于 MySQL 解释`TIME`值表示经过的时间而不是一天中的时间。

*`expr`*被视为字符串，因此如果您在`INTERVAL`中指定非字符串值，请小心。例如，使用`HOUR_MINUTE`的间隔符号，'6/4'被视为 6 小时 4 分钟，而`6/4`计算结果为`1.5000`，被视为 1 小时 5000 分钟：

```sql
mysql> SELECT '6/4', 6/4;
 -> 1.5000
mysql> SELECT DATE_ADD('2019-01-01', INTERVAL '6/4' HOUR_MINUTE);
 -> '2019-01-01 06:04:00'
mysql> SELECT DATE_ADD('2019-01-01', INTERVAL 6/4 HOUR_MINUTE);
 -> '2019-01-04 12:20:00'
```

为了确保间隔值的解释符合您的期望，可以使用`CAST()`操作。将`6/4`视为 1 小时 5 分钟，将其转换为具有单个小数位的`DECIMAL` - DECIMAL, NUMERIC")值：

```sql
mysql> SELECT CAST(6/4 AS DECIMAL(3,1));
 -> 1.5
mysql> SELECT DATE_ADD('1970-01-01 12:00:00',
 ->                 INTERVAL CAST(6/4 AS DECIMAL(3,1)) HOUR_MINUTE);
 -> '1970-01-01 13:05:00'
```

如果您向日期值添加或减去包含时间部分的内容，则结果会自动转换为日期时间值：

```sql
mysql> SELECT DATE_ADD('2023-01-01', INTERVAL 1 DAY);
 -> '2023-01-02'
mysql> SELECT DATE_ADD('2023-01-01', INTERVAL 1 HOUR);
 -> '2023-01-01 01:00:00'
```

如果您添加`MONTH`、`YEAR_MONTH`或`YEAR`，并且结果日期的天数大于新月份的最大天数，则天数会调整为新月份的最大天数：

```sql
mysql> SELECT DATE_ADD('2019-01-30', INTERVAL 1 MONTH);
 -> '2019-02-28'
```

日期算术运算需要完整的日期，不适用于不完整的日期，如`'2016-07-00'`或格式错误的日期：

```sql
mysql> SELECT DATE_ADD('2016-07-00', INTERVAL 1 DAY);
 -> NULL
mysql> SELECT '2005-03-32' + INTERVAL 1 MONTH;
 -> NULL
```
