# 13.2.1 日期和时间数据类型语法

> 原文：[`dev.mysql.com/doc/refman/8.0/en/date-and-time-type-syntax.html`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-type-syntax.html)

用于表示时间值的日期和时间数据类型是 `DATE`、`TIME`、`DATETIME`、`TIMESTAMP` 和 `YEAR`。

对于 `DATE` 和 `DATETIME` 范围描述，“支持” 表示虽然较早的值可能有效，但不保证。

MySQL 允许 `TIME`、`DATETIME` 和 `TIMESTAMP` 值具有微秒（6 位数字）精度的小数秒。要定义包含小数秒部分的列，请使用语法 `*`type_name`*(*`fsp`*)`，其中 *`type_name`* 是 `TIME`、`DATETIME` 或 `TIMESTAMP`，*`fsp`* 是小数秒精度。例如：

```sql
CREATE TABLE t1 (t TIME(3), dt DATETIME(6), ts TIMESTAMP(0));
```

*`fsp`* 值（如果提供）必须在 0 到 6 的范围内。值为 0 表示没有小数部分。如果省略，则默认精度为 0。（这与标准 SQL 默认值 6 不同，以保持与之前 MySQL 版本的兼容性。）

表中的任何 `TIMESTAMP` 或 `DATETIME` 列都可以具有自动初始化和更新属性；请参阅 第 13.2.5 节，“TIMESTAMP 和 DATETIME 的自动初始化和更新”。

+   `DATE`

    日期。支持的范围是 `'1000-01-01'` 到 `'9999-12-31'`。MySQL 以 `'*`YYYY-MM-DD`*'` 格式显示 `DATE` 值，但允许将值分配给 `DATE` 列，使用字符串或数字。

+   [`DATETIME[(*`fsp`*)]`](datetime.html "13.2.2 DATE、DATETIME 和 TIMESTAMP 类型")

    日期和时间的组合。支持范围为`'1000-01-01 00:00:00.000000'`到`'9999-12-31 23:59:59.499999'`。MySQL 以`'*`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*]'`格式显示`DATETIME`值，但允许将值分配给`DATETIME`列，使用字符串或数字。

    可以给出范围从 0 到 6 的可选*`fsp`*值，以指定小数秒精度。值为 0 表示没有小数部分。如果省略，则默认精度为 0。

    自动初始化和更新`DATETIME`列到当前日期和时间可以使用`DEFAULT`和`ON UPDATE`列定义子句来指定，如第 13.2.5 节，“TIMESTAMP 和 DATETIME 的自动初始化和更新”中所述。

+   [`TIMESTAMP[(*`fsp`*)]`](datetime.html "13.2.2 日期、DATETIME 和 TIMESTAMP 类型")

    时间戳。范围为`'1970-01-01 00:00:01.000000'` UTC 到`'2038-01-19 03:14:07.499999'` UTC。`TIMESTAMP`值存储为自纪元（`'1970-01-01 00:00:00'` UTC）以来的秒数。`TIMESTAMP`不能表示值`'1970-01-01 00:00:00'`，因为那相当于自纪元以来的 0 秒，值 0 保留用于表示`'0000-00-00 00:00:00'`，即“零”`TIMESTAMP`值。

    可以给出范围从 0 到 6 的可选*`fsp`*值，以指定小数秒精度。值为 0 表示没有小数部分。如果省略，则默认精度为 0。

    服务器处理`TIMESTAMP`定义的方式取决于`explicit_defaults_for_timestamp`系统变量的值（参见第 7.1.8 节，“服务器系统变量”）。

    如果启用`explicit_defaults_for_timestamp`，则不会自动将`DEFAULT CURRENT_TIMESTAMP`或`ON UPDATE CURRENT_TIMESTAMP`属性分配给任何`TIMESTAMP`列。它们必须明确包含在列定义中。此外，任何未明确声明为`NOT NULL`的`TIMESTAMP`允许`NULL`值。

    如果禁用`explicit_defaults_for_timestamp`，服务器处理`TIMESTAMP`如下：

    除非另有说明，否则表中的第一个`TIMESTAMP`列被定义为在未明确分配值的情况下自动设置为最近修改的日期和时间。这使得`TIMESTAMP`对于记录`INSERT`或`UPDATE`操作的时间戳非常有用。您还可以通过将其分配为`NULL`值将任何`TIMESTAMP`列设置为当前日期和时间，除非已使用`NULL`属性定义允许`NULL`值。

    可以使用`DEFAULT CURRENT_TIMESTAMP`和`ON UPDATE CURRENT_TIMESTAMP`列定义子句指定自动初始化和更新为当前日期和时间。默认情况下，第一个`TIMESTAMP`列具有这些属性，如前所述。但是，表中的任何`TIMESTAMP`列都可以定义为具有这些属性。

+   [`TIME[(*`fsp`*)]`](time.html "13.2.3 时间类型")

    一个时间。范围是`'-838:59:59.000000'`到`'838:59:59.000000'`。MySQL 以`'*`hh:mm:ss`*[.*`fraction`*]'`格式显示`TIME`值，但允许使用字符串或数字将值分配给`TIME`列。

    可以给出范围为 0 到 6 的可选*`fsp`*值以指定小数秒精度。值为 0 表示没有小数部分。如果省略，则默认精度为 0。

+   [`YEAR[(4)]`](year.html "13.2.4 年份类型")

    以 4 位数字格式表示的年份。MySQL 以*`YYYY`*格式显示`YEAR`值，但允许使用字符串或数字将值分配给`YEAR`列。值显示为`1901`到`2155`，或`0000`。

    有关`YEAR`显示格式和输入值解释的其他信息，请参见第 13.2.4 节，“年份类型”。

    注意

    截至 MySQL 8.0.19，带有显式显示宽度的`YEAR(4)`数据类型已被弃用；您应该期望在未来的 MySQL 版本中删除对其的支持。取而代之，请使用没有显示宽度的`YEAR`，其含义相同。

    MySQL 8.0 不支持旧版本 MySQL 中允许的 2 位数`YEAR(2)`数据类型。有关转换为 4 位数`YEAR`的说明，请参见 2-Digit YEAR(2) Limitations and Migrating to 4-Digit YEAR，在 MySQL 5.7 参考手册中。

`SUM()`和`AVG()`聚合函数不适用于时间值。（它们将值转换为数字，丢失第一个非数字字符后的所有内容。）为解决此问题，需将其转换为数字单位，执行聚合操作，然后再转换回时间值。示例：

```sql
SELECT SEC_TO_TIME(SUM(TIME_TO_SEC(*time_col*))) FROM *tbl_name*;
SELECT FROM_DAYS(SUM(TO_DAYS(*date_col*))) FROM *tbl_name*;
```
