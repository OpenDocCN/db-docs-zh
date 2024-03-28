# 14.20.3 窗口函数帧规范

> 原文：[`dev.mysql.com/doc/refman/8.0/en/window-functions-frames.html`](https://dev.mysql.com/doc/refman/8.0/en/window-functions-frames.html)

与窗口函数一起使用的窗口的定义可以包括一个帧子句。帧是当前分区的子集，帧子句指定如何定义子集。

帧是相对于当前行确定的，这使得帧可以根据当前行在其分区中的位置移动。例如：

+   通过将帧定义为从分区开始到当前行的所有行，您可以为每行计算累计总和。

+   通过将帧定义为在当前行的两侧扩展*`N`*行，您可以计算滚动平均值。

以下查询演示了使用移动帧来计算每组时间排序的`level`值内的累计总和，以及从当前行和紧随其后的行计算的滚动平均值：

```sql
mysql> SELECT
         time, subject, val,
         SUM(val) OVER (PARTITION BY subject ORDER BY time
                        ROWS UNBOUNDED PRECEDING)
           AS running_total,
         AVG(val) OVER (PARTITION BY subject ORDER BY time
                        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)
           AS running_average
       FROM observations;
+----------+---------+------+---------------+-----------------+
| time     | subject | val  | running_total | running_average |
+----------+---------+------+---------------+-----------------+
| 07:00:00 | st113   |   10 |            10 |          9.5000 |
| 07:15:00 | st113   |    9 |            19 |         14.6667 |
| 07:30:00 | st113   |   25 |            44 |         18.0000 |
| 07:45:00 | st113   |   20 |            64 |         22.5000 |
| 07:00:00 | xh458   |    0 |             0 |          5.0000 |
| 07:15:00 | xh458   |   10 |            10 |          5.0000 |
| 07:30:00 | xh458   |    5 |            15 |         15.0000 |
| 07:45:00 | xh458   |   30 |            45 |         20.0000 |
| 08:00:00 | xh458   |   25 |            70 |         27.5000 |
+----------+---------+------+---------------+-----------------+
```

对于`running_average`列，第一个和最后一个之后没有帧行。在这些情况下，`AVG()`计算可用行的平均值。

作为窗口函数使用的聚合函数在当前行帧上操作，这些非聚合窗口函数也是如此：

```sql
FIRST_VALUE()
LAST_VALUE()
NTH_VALUE()
```

标准 SQL 指定对整个分区操作的窗口函数不应具有帧子句。MySQL 允许这些函数具有帧子句，但会忽略它。即使指定了帧，这些函数也使用整个分区：

```sql
CUME_DIST()
DENSE_RANK()
LAG()
LEAD()
NTILE()
PERCENT_RANK()
RANK()
ROW_NUMBER()
```

如果提供了帧子句，则具有以下语法：

```sql
*frame_clause*:
    *frame_units* *frame_extent*

*frame_units*:
    {ROWS | RANGE}
```

在没有帧子句的情况下，默认帧取决于是否存在`ORDER BY`子句，如本节后面所述。

*`frame_units`*值表示当前行与帧行之间的关系类型：

+   `ROWS`: 帧由开始和结束行位置定义。偏移量是当前行号与行号之间的差异。

+   `RANGE`: 帧由值范围内的行定义。偏移量是当前行值与行值之间的差异。

*`frame_extent`*值表示帧的起始点和结束点。您可以仅指定帧的起始点（在这种情况下，当前行隐含为结束点），或使用`BETWEEN`指定帧的两个端点：

```sql
*frame_extent*:
    {*frame_start* | *frame_between*}

*frame_between*:
    BETWEEN *frame_start* AND *frame_end*

*frame_start*, *frame_end*: {
    CURRENT ROW
  | UNBOUNDED PRECEDING
  | UNBOUNDED FOLLOWING
  | *expr* PRECEDING
  | *expr* FOLLOWING
}
```

使用`BETWEEN`语法，*`frame_start`*不能出现在*`frame_end`*之后。

允许的*`frame_start`*和*`frame_end`*值具有以下含义：

+   `CURRENT ROW`: 对于`ROWS`，边界是当前行。对于`RANGE`，边界是当前行的对等行。

+   `UNBOUNDED PRECEDING`: 边界是第一个分区行。

+   `UNBOUNDED FOLLOWING`: 边界是最后一个分区行。

+   `*`expr`* PRECEDING`: 对于`ROWS`，边界是当前行之前的*`expr`*行。对于`RANGE`，边界是具有值等于当前行值减去*`expr`*的行；如果当前行值为`NULL`，则边界是该行的对等行。

    对于`*`expr`* PRECEDING`（和`*`expr`* FOLLOWING`），*`expr`*可以是一个`?`参数标记（用于准备的语句中），一个非负数数字文字，或者形式为`INTERVAL *`val`* *`unit`*`的时间间隔。对于`INTERVAL`表达式，*`val`*指定非负的间隔值，*`unit`*是一个关键字，指示值应该以哪种单位解释。（有关允许的*`units`*说明符的详细信息，请参阅第 14.7 节“日期和时间函数”中的`DATE_ADD()`函数的描述。）

    在数字或时间*`expr`*上的`RANGE`需要在数字或时间表达式上使用`ORDER BY`。

    有效的`*`expr`* PRECEDING`和`*`expr`* FOLLOWING`指示的示例：

    ```sql
    10 PRECEDING
    INTERVAL 5 DAY PRECEDING
    5 FOLLOWING
    INTERVAL '2:30' MINUTE_SECOND FOLLOWING
    ```

+   `*`expr`* FOLLOWING`: 对于`ROWS`，边界是当前行之后的*`expr`*行。对于`RANGE`，边界是具有值等于当前行值加上*`expr`*的行；如果当前行值为`NULL`，则边界是该行的对等行。

    对于*`expr`*的允许值，请参阅`*`expr`* PRECEDING`的描述。

以下查询演示了`FIRST_VALUE()`，`LAST_VALUE()`和两个`NTH_VALUE()`实例：

```sql
mysql> SELECT
         time, subject, val,
         FIRST_VALUE(val)  OVER w AS 'first',
         LAST_VALUE(val)   OVER w AS 'last',
         NTH_VALUE(val, 2) OVER w AS 'second',
         NTH_VALUE(val, 4) OVER w AS 'fourth'
       FROM observations
       WINDOW w AS (PARTITION BY subject ORDER BY time
                    ROWS UNBOUNDED PRECEDING);
+----------+---------+------+-------+------+--------+--------+
| time     | subject | val  | first | last | second | fourth |
+----------+---------+------+-------+------+--------+--------+
| 07:00:00 | st113   |   10 |    10 |   10 |   NULL |   NULL |
| 07:15:00 | st113   |    9 |    10 |    9 |      9 |   NULL |
| 07:30:00 | st113   |   25 |    10 |   25 |      9 |   NULL |
| 07:45:00 | st113   |   20 |    10 |   20 |      9 |     20 |
| 07:00:00 | xh458   |    0 |     0 |    0 |   NULL |   NULL |
| 07:15:00 | xh458   |   10 |     0 |   10 |     10 |   NULL |
| 07:30:00 | xh458   |    5 |     0 |    5 |     10 |   NULL |
| 07:45:00 | xh458   |   30 |     0 |   30 |     10 |     30 |
| 08:00:00 | xh458   |   25 |     0 |   25 |     10 |     30 |
+----------+---------+------+-------+------+--------+--------+
```

每个函数使用当前帧中的行，根据所示的窗口定义，该帧从第一个分区行延伸到当前行。对于`NTH_VALUE()`调用，当前帧并不总是包括请求的行；在这种情况下，返回值为`NULL`。

在没有帧子句的情况下，默认帧取决于是否存在`ORDER BY`子句：

+   使用`ORDER BY`：默认帧包括从分区开始到当前行的所有行，包括当前行的所有对等行（根据`ORDER BY`子句与当前行相等的行）。默认等同于此帧规范：

    ```sql
    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ```

+   没有`ORDER BY`：默认帧包括所有分区行（因为没有`ORDER BY`，所有分区行都是对等的）。默认等同于此帧规范：

    ```sql
    RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ```

因为默认帧取决于是否存在`ORDER BY`，为了获得确定性结果，向查询添加`ORDER BY`可能会改变结果。（例如，`SUM()`产生的值可能会改变。）为了获得相同的结果但按`ORDER BY`排序，提供一个明确的帧规范，无论是否存在`ORDER BY`都会使用。

当当前行值为`NULL`时，框架规范的含义可能不明显。假设是这种情况，以下示例说明了各种框架规范的应用：

+   `ORDER BY X ASC RANGE BETWEEN 10 FOLLOWING AND 15 FOLLOWING`

    框架从`NULL`开始，止于`NULL`，因此只包括值为`NULL`的行。

+   `ORDER BY X ASC RANGE BETWEEN 10 FOLLOWING AND UNBOUNDED FOLLOWING`

    框架从`NULL`开始，止于分区末尾。因为`ASC`排序将`NULL`值放在最前面，所以框架是整个分区。

+   `ORDER BY X DESC RANGE BETWEEN 10 FOLLOWING AND UNBOUNDED FOLLOWING`

    框架从`NULL`开始，止于分区末尾。因为`DESC`排序将`NULL`值放在最后，所以框架只包括`NULL`值。

+   `ORDER BY X ASC RANGE BETWEEN 10 PRECEDING AND UNBOUNDED FOLLOWING`

    框架从`NULL`开始，止于分区末尾。因为`ASC`排序将`NULL`值放在最前面，所以框架是整个分区。

+   `ORDER BY X ASC RANGE BETWEEN 10 PRECEDING AND 10 FOLLOWING`

    框架从`NULL`开始，止于`NULL`，因此只包括值为`NULL`的行。

+   `ORDER BY X ASC RANGE BETWEEN 10 PRECEDING AND 1 PRECEDING`

    框架从`NULL`开始，止于`NULL`，因此只包括值为`NULL`的行。

+   `ORDER BY X ASC RANGE BETWEEN UNBOUNDED PRECEDING AND 10 FOLLOWING`

    框架从分区开始，止于值为`NULL`的行。因为`ASC`排序将`NULL`值放在最前面，所以框架只包括`NULL`值。
