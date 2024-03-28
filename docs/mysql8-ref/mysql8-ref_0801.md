# 14.7 日期和时间函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html)

本节描述了用于操作时间值的函数。有关每种日期和时间类型的值范围以及可以指定值的有效格式的描述，请参见第 13.2 节，“日期和时间数据类型”。

**表格 14.11 日期和时间函数**

| 名称 | 描述 |
| --- | --- |
| `ADDDATE()` | 将时间值（间隔）添加到日期值 |
| `ADDTIME()` | 添加时间 |
| `CONVERT_TZ()` | 将一个时区转换为另一个时区 |
| `CURDATE()` | 返回当前日期 |
| `CURRENT_DATE()`, `CURRENT_DATE` | CURDATE()的同义词 |
| `CURRENT_TIME()`, `CURRENT_TIME` | CURTIME()的同义词 |
| `CURRENT_TIMESTAMP()`, `CURRENT_TIMESTAMP` | NOW()的同义词 |
| `CURTIME()` | 返回当前时间 |
| `DATE()` | 提取日期或日期时间表达式的日期部分 |
| `DATE_ADD()` | 将时间值（间隔）添加到日期值 |
| `DATE_FORMAT()` | 格式化指定的日期 |
| `DATE_SUB()` | 从日期中减去一个时间值（间隔） |
| `DATEDIFF()` | 计算两个日期之间的差值 |
| `DAY()` | DAYOFMONTH()的同义词 |
| `DAYNAME()` | 返回星期几的名称 |
| `DAYOFMONTH()` | 返回月份中的日期（0-31） |
| `DAYOFWEEK()` | 返回参数的星期索引 |
| `DAYOFYEAR()` | 返回一年中的日期（1-366） |
| `EXTRACT()` | 提取日期的部分 |
| `FROM_DAYS()` | 将天数转换为日期 |
| `FROM_UNIXTIME()` | 将 Unix 时间戳格式化为日期 |
| `GET_FORMAT()` | 返回日期格式字符串 |
| `HOUR()` | 提取小时 |
| `LAST_DAY` | 返回参数月份的最后一天 |
| `LOCALTIME()`, `LOCALTIME` | NOW()的同义词 |
| `LOCALTIMESTAMP`, `LOCALTIMESTAMP()` | NOW()的同义词 |
| `MAKEDATE()` | 从年份和一年中的天数创建日期 |
| `MAKETIME()` | 从小时、分钟、秒创建时间 |
| `MICROSECOND()` | 返回参数的微秒 |
| `MINUTE()` | 返回参数的分钟 |
| `MONTH()` | 返回传递日期的月份 |
| `MONTHNAME()` | 返回月份的名称 |
| `NOW()` | 返回当前日期和时间 |
| `PERIOD_ADD()` | 向年-月添加一个周期 |
| `PERIOD_DIFF()` | 返回两个周期之间的月数 |
| `QUARTER()` | 返回日期参数的季度 |
| `SEC_TO_TIME()` | 将秒转换为'hh:mm:ss'格式 |
| `SECOND()` | 返回秒数（0-59） |
| `STR_TO_DATE()` | 将字符串转换为日期 |
| `SUBDATE()` | 在使用三个参数调用时是 DATE_SUB()的同义词 |
| `SUBTIME()` | 时间相减 |
| `SYSDATE()` | 返回函数执行时的时间 |
| `TIME()` | 提取传递表达式的时间部分 |
| `TIME_FORMAT()` | 格式化为时间 |
| `TIME_TO_SEC()` | 返回转换为秒的参数 |
| `TIMEDIFF()` | 时间相减 |
| `TIMESTAMP()` | 使用单个参数，此函数返回日期或日期时间表达式；使用两个参数，返回参数的总和 |
| `TIMESTAMPADD()` | 向日期时间表达式添加一个间隔 |
| `TIMESTAMPDIFF()` | 返回两个日期时间表达式的差异，使用指定的单位 |
| `TO_DAYS()` | 返回转换为天数的日期参数 |
| `TO_SECONDS()` | 返回自公元 0 年以来的秒数 |
| `UNIX_TIMESTAMP()` | 返回 Unix 时间戳 |
| `UTC_DATE()` | 返回当前的 UTC 日期 |
| `UTC_TIME()` | 返回当前的 UTC 时间 |
| `UTC_TIMESTAMP()` | 返回当前的 UTC 日期和时间 |
| `WEEK()` | 返回周数 |
| `WEEKDAY()` | 返回工作日索引 |
| `WEEKOFYEAR()` | 返回日期的日历周（1-53） |
| `YEAR()` | 返回年份 |
| `YEARWEEK()` | 返回年份和周数 |
| 名称 | 描述 |

以下是一个使用日期函数的示例。以下查询选择所有*`date_col`*值在过去 30 天内的行：

```sql
mysql> SELECT *something* FROM *tbl_name*
 -> WHERE DATE_SUB(CURDATE(),INTERVAL 30 DAY) <= *date_col*;
```

查询还选择未来日期的行。

函数通常接受日期值，但会忽略时间部分。通常接受时间值的函数会接受日期时间值并忽略日期部分。

返回当前日期或时间的函数在每次查询执行开始时仅计算一次。这意味着在单个查询中多次引用诸如`NOW()`的函数总是产生相同的结果。（对于我们的目的，单个查询还包括对存储程序（存储过程、触发器或事件）的调用以及该程序调用的所有子程序。）这个原则也适用于`CURDATE()`、`CURTIME()`、`UTC_DATE()`、`UTC_TIME()`、`UTC_TIMESTAMP()`以及它们的任何同义词。

`CURRENT_TIMESTAMP()`, `CURRENT_TIME()`, `CURRENT_DATE()`和`FROM_UNIXTIME()`函数返回当前会话时区的值，该时区作为`time_zone`系统变量的会话值可用。此外，`UNIX_TIMESTAMP()`假定其参数是会话时区中的日期时间值。参见 Section 7.1.15, “MySQL Server Time Zone Support”。

一些日期函数可以与“零”日期或不完整日期一起使用，例如`'2001-11-00'`，而其他函数则不能。通常用于提取日期部分的函数可以处理不完整日期，因此在其他情况下可能会返回 0 而不是非零值。例如：

```sql
mysql> SELECT DAYOFMONTH('2001-11-00'), MONTH('2005-00-00');
 -> 0, 0
```

其他函数期望完整日期并对不完整日期返回`NULL`。这些函数包括执行日期运算或将日期部分映射到名称的函数。例如：

```sql
mysql> SELECT DATE_ADD('2006-05-00',INTERVAL 1 DAY);
 -> NULL
mysql> SELECT DAYNAME('2006-05-00');
 -> NULL
```

当传递`DATE()`函数值作为参数时，一些函数是严格的，并拒绝具有零天部分的不完整日期：`CONVERT_TZ()`, `DATE_ADD()`, `DATE_SUB()`, `DAYOFYEAR()`, `TIMESTAMPDIFF()`, `TO_DAYS()`, `TO_SECONDS()`, `WEEK()`, `WEEKDAY()`, `WEEKOFYEAR()`, `YEARWEEK()`.

支持`TIME`、`DATETIME`和`TIMESTAMP`值的分数秒，精度可达微秒。接受时间参数的函数接受具有分数秒的值。从时间函数返回的值包括适当的分数秒。

+   `ADDDATE(*`date`*,INTERVAL *`expr`* *`unit`*)`, `ADDDATE(*`date`*,*`days`*)`

    当以第二个参数的`INTERVAL`形式调用时，`ADDDATE()`是`DATE_ADD()`的同义词。相关函数`SUBDATE()`是`DATE_SUB()`的同义词。有关`INTERVAL` *`unit`*参数的信息，请参见时间间隔。

    ```sql
    mysql> SELECT DATE_ADD('2008-01-02', INTERVAL 31 DAY);
     -> '2008-02-02'
    mysql> SELECT ADDDATE('2008-01-02', INTERVAL 31 DAY);
     -> '2008-02-02'
    ```

    当以第二个参数的*`days`*形式调用时，MySQL 将其视为要添加到*`expr`*的整数天数。

    ```sql
    mysql> SELECT ADDDATE('2008-01-02', 31);
     -> '2008-02-02'
    ```

    如果*`date`*或*`days`*为`NULL`，此函数返回`NULL`。

+   `ADDTIME(*`expr1`*,*`expr2`*)`

    `ADDTIME()`将*`expr2`*添加到*`expr1`*并返回结果。*`expr1`*是时间或日期时间表达式，*`expr2`*是时间表达式。如果*`expr1`*或*`expr2`*为`NULL`，则返回`NULL`。

    从 MySQL 8.0.28 开始，此函数和`SUBTIME()`函数的返回类型如下确定：

    +   如果第一个参数是动态参数（例如在准备好的语句中），返回类型为`TIME`。

    +   否则，函数的解析类型源自第一个参数的解析类型。

    ```sql
    mysql> SELECT ADDTIME('2007-12-31 23:59:59.999999', '1 1:1:1.000002');
     -> '2008-01-02 01:01:01.000001'
    mysql> SELECT ADDTIME('01:00:00.999999', '02:00:00.999998');
     -> '03:00:01.999997'
    ```

+   `CONVERT_TZ(*`dt`*,*`from_tz`*,*`to_tz`*)`

    `CONVERT_TZ()`将给定时区*`from_tz`*的日期时间值*`dt`*转换为给定时区*`to_tz`*的值并返回结果。时区的指定方式如第 7.1.15 节“MySQL 服务器时区支持”中所述。如果任何参数无效或任何参数为`NULL`，此函数返回`NULL`。

    在 32 位平台上，此函数的支持值范围与`TIMESTAMP`类型相同（有关范围信息，请参见第 13.2.1 节“日期和时间数据类型语法”）。在 64 位平台上，从 MySQL 8.0.28 开始，最大支持值为`'3001-01-18 23:59:59.999999'` UTC。

    无论平台或 MySQL 版本如何，如果从*`from_tz`*转换为 UTC 时的值超出支持范围，则不进行转换。

    ```sql
    mysql> SELECT CONVERT_TZ('2004-01-01 12:00:00','GMT','MET');
     -> '2004-01-01 13:00:00'
    mysql> SELECT CONVERT_TZ('2004-01-01 12:00:00','+00:00','+10:00');
     -> '2004-01-01 22:00:00'
    ```

    注意

    要使用诸如`'MET'`或`'Europe/Amsterdam'`之类的命名时区，必须正确设置时区表。有关说明，请参见第 7.1.15 节“MySQL 服务器时区支持”。

+   `CURDATE()`

    返回当前日期作为值，格式为`'*`YYYY-MM-DD`*'`或*`YYYYMMDD`*，取决于函数在字符串或数字上下文中的使用方式。

    ```sql
    mysql> SELECT CURDATE();
     -> '2008-06-13'
    mysql> SELECT CURDATE() + 0;
     -> 20080613
    ```

+   `CURRENT_DATE`, `CURRENT_DATE()`

    `CURRENT_DATE`和`CURRENT_DATE()`是`CURDATE()`的同义词。

+   `CURRENT_TIME`, [`CURRENT_TIME([*`fsp`*])`](date-and-time-functions.html#function_current-time)

    `CURRENT_TIME`和`CURRENT_TIME()`是`CURTIME()`的同义词。

+   `CURRENT_TIMESTAMP`, [`CURRENT_TIMESTAMP([*`fsp`*])`](date-and-time-functions.html#function_current-timestamp)

    `CURRENT_TIMESTAMP`和`CURRENT_TIMESTAMP()`是`NOW()`的同义词。

+   [`CURTIME([*`fsp`*])`](date-and-time-functions.html#function_curtime)

    返回当前时间作为值，格式为*`'hh:mm:ss'`*或*`hhmmss`*，取决于函数在字符串或数字上下文中的使用方式。该值以会话时区表示。

    如果给定*`fsp`*参数以指定从 0 到 6 的小数秒精度，则返回值包括相应数量的小数秒部分。

    ```sql
    mysql> SELECT CURTIME();
    +-----------+
    | CURTIME() |
    +-----------+
    | 19:25:37  |
    +-----------+

    mysql> SELECT CURTIME() + 0;
    +---------------+
    | CURTIME() + 0 |
    +---------------+
    |        192537 |
    +---------------+

    mysql> SELECT CURTIME(3);
    +--------------+
    | CURTIME(3)   |
    +--------------+
    | 19:25:37.840 |
    +--------------+
    ```

+   `DATE(*`expr`*)`

    提取日期或日期时间表达式*`expr`*的日期部分。如果*`expr`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT DATE('2003-12-31 01:02:03');
     -> '2003-12-31'
    ```

+   `DATEDIFF(*`expr1`*,*`expr2`*)`

    `DATEDIFF()` 返回*`expr1`*和*`expr2`*之间相差的天数值，以天数表示从一个日期到另一个日期。*`expr1`*和*`expr2`*是日期或日期时间表达式。计算中仅使用值的日期部分。

    ```sql
    mysql> SELECT DATEDIFF('2007-12-31 23:59:59','2007-12-30');
     -> 1
    mysql> SELECT DATEDIFF('2010-11-30 23:59:59','2010-12-31');
     -> -31
    ```

    如果*`expr1`*或*`expr2`*为`NULL`，此函数返回`NULL`。

+   `DATE_ADD(*`date`*,INTERVAL *`expr`* *`unit`*)`, `DATE_SUB(*`date`*,INTERVAL *`expr`* *`unit`*)`

    这些函数执行日期算术运算。*`date`*参数指定起始日期或日期时间值。*`expr`*是指定要从起始日期中添加或减去的间隔值的表达式。*`expr`*被评估为字符串；它可以以`-`开头表示负间隔。*`unit`*是指示应解释表达式的单位的关键字。

    有关时间间隔语法的更多信息，包括完整的*`unit`*指定符列表，每个*`unit`*值的*`expr`*参数的预期形式，以及在时间算术中操作数解释的规则，请参阅时间间隔。

    返回值取决于参数：

    +   如果*`date`*为`NULL`，函数将返回`NULL`。

    +   如果*`date`*参数是`DATE`值，并且您的计算仅涉及`YEAR`、`MONTH`和`DAY`部分（即没有时间部分），则返回`DATE`。

    +   （*MySQL 8.0.28 及更高版本*：）如果*`date`*参数是`TIME`值，并且计算仅涉及`HOURS`、`MINUTES`和`SECONDS`部分（即没有日期部分），则返回`TIME`。

    +   如果第一个参数是`DATETIME`（或`TIMESTAMP`）值，或者第一个参数是`DATE`且*`unit`*值使用`HOURS`、`MINUTES`或`SECONDS`，或者第一个参数是`TIME`且*`unit`*值使用`YEAR`、`MONTH`或`DAY`，则返回`DATETIME`。

    +   （*MySQL 8.0.28 及更高版本*：）如果第一个参数是动态参数（例如，准备语句的参数），且第二个参数是仅包含`YEAR`、`MONTH`或`DAY`值组合的间隔，则其解析类型为`DATE`；否则，其类型为`DATETIME`。

    +   否则为字符串（类型`VARCHAR`）。

    注意

    在 MySQL 8.0.22 至 8.0.27 中，在准备语句中使用时，这些函数无论参数类型如何都返回`DATETIME`值。（Bug #103781）

    为了确保结果是`DATETIME`，您可以使用`CAST()`将第一个参数转换为`DATETIME`。

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

    当向`DATE`或`DATETIME`值添加`MONTH`间隔时，并且结果日期包含给定月份中不存在的日期时，日期将调整为该月的最后一天，如下所示：

    ```sql
    mysql> SELECT DATE_ADD('2024-03-30', INTERVAL 1 MONTH) AS d1, 
         >        DATE_ADD('2024-03-31', INTERVAL 1 MONTH) AS d2;
    +------------+------------+
    | d1         | d2         |
    +------------+------------+
    | 2024-04-30 | 2024-04-30 |
    +------------+------------+
    1 row in set (0.00 sec)
    ```

+   `DATE_FORMAT(*`date`*,*`format`*)`

    根据*`format`*字符串格式化*`date`*值。如果任一参数为`NULL`，函数将返回`NULL`。

    下表中显示的指示符可用于 *`format`* 字符串。在格式指示符字符之前需要 `%` 字符。这些指示符也适用于其他函数：`STR_TO_DATE()`, `TIME_FORMAT()`, `UNIX_TIMESTAMP()`。

    | 指示符 | 描述 |
    | --- | --- |
    | `%a` | 缩写星期名称 (`Sun`..`Sat`) |
    | `%b` | 缩写月份名称 (`Jan`..`Dec`) |
    | `%c` | 月份，数字 (`0`..`12`) |
    | `%D` | 带有英文后缀的日期 (`0th`, `1st`, `2nd`, `3rd`, …) |
    | `%d` | 日期，数字 (`00`..`31`) |
    | `%e` | 日期，数字 (`0`..`31`) |
    | `%f` | 微秒 (`000000`..`999999`) |
    | `%H` | 小时 (`00`..`23`) |
    | `%h` | 小时 (`01`..`12`) |
    | `%I` | 小时 (`01`..`12`) |
    | `%i` | 分钟，数字 (`00`..`59`) |
    | `%j` | 一年中的日期 (`001`..`366`) |
    | `%k` | 小时 (`0`..`23`) |
    | `%l` | 小时 (`1`..`12`) |
    | `%M` | 月份名称 (`January`..`December`) |
    | `%m` | 月份，数字 (`00`..`12`) |
    | `%p` | `AM` 或 `PM` |
    | `%r` | 时间，12 小时制 (*`hh:mm:ss`* 后跟 `AM` 或 `PM`) |
    | `%S` | 秒数 (`00`..`59`) |
    | `%s` | 秒数 (`00`..`59`) |
    | `%T` | 时间，24 小时制 (*`hh:mm:ss`*) |
    | `%U` | 周数 (`00`..`53`), 星期日为一周的第一天; `WEEK()` 模式 0 |
    | `%u` | 周数 (`00`..`53`), 星期一为一周的第一天; `WEEK()` 模式 1 |
    | `%V` | 周数 (`01`..`53`), 星期日为一周的第一天; `WEEK()` 模式 2; 与 `%X` 一起使用 |
    | `%v` | 周数 (`01`..`53`), 星期一为一周的第一天; `WEEK()` 模式 3; 与 `%x` 一起使用 |
    | `%W` | 星期名称 (`Sunday`..`Saturday`) |
    | `%w` | 星期几 (`0`=星期日..`6`=星期六) |
    | `%X` | 一周的年份，星期日为一周的第一天，数字，四位数; 与 `%V` 一起使用 |
    | `%x` | 一周的年份，星期一为一周的第一天，数字，四位数; 与 `%v` 一起使用 |
    | `%Y` | 年份，数字，四位数 |
    | `%y` | 年份，数字 (两位数) |
    | `%%` | 一个字面上的 `%` 字符 |
    | `%*`x`*` | *`x`*，对于上面未列出的任何“*`x`*” |
    | 指示符 | 描述 |

    月份和日期指示符的范围从零开始，因为 MySQL 允许存储不完整的日期，如 `'2014-00-00'`。

    用于日期和月份名称和缩写的语言由 `lc_time_names` 系统变量的值控制（第 12.16 节，“MySQL 服务器区域设置支持”）。

    对于`%U`、`%u`、`%V`和`%v`格式说明符，请参阅`WEEK()`函数的描述，了解有关模式值的信息。模式影响周编号的方式。

    `DATE_FORMAT()`返回一个字符串，其中包含由`character_set_connection`和`collation_connection`给定的字符集和校对规则，以便返回包含非 ASCII 字符的月份和星期几名称。

    ```sql
    mysql> SELECT DATE_FORMAT('2009-10-04 22:23:00', '%W %M %Y');
     -> 'Sunday October 2009'
    mysql> SELECT DATE_FORMAT('2007-10-04 22:23:00', '%H:%i:%s');
     -> '22:23:00'
    mysql> SELECT DATE_FORMAT('1900-10-04 22:23:00',
     ->                 '%D %y %a %d %m %b %j');
     -> '4th 00 Thu 04 10 Oct 277'
    mysql> SELECT DATE_FORMAT('1997-10-04 22:23:00',
     ->                 '%H %k %I %r %T %S %w');
     -> '22 22 10 10:23:00 PM 22:23:00 00 6'
    mysql> SELECT DATE_FORMAT('1999-01-01', '%X %V');
     -> '1998 52'
    mysql> SELECT DATE_FORMAT('2006-06-00', '%d');
     -> '00'
    ```

+   `DATE_SUB(*`date`*,INTERVAL *`expr`* *`unit`*)`

    查看`DATE_ADD()`的描述。

+   `DAY(*`date`*)`

    `DAY()`是`DAYOFMONTH()`的同义词。

+   `DAYNAME(*`date`*)`

    返回*`date`*的星期几名称。名称所使用的语言由`lc_time_names`系统变量的值控制（参见第 12.16 节，“MySQL 服务器区域设置支持”）。如果*`date`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT DAYNAME('2007-02-03');
     -> 'Saturday'
    ```

+   `DAYOFMONTH(*`date`*)`

    返回*`date`*的月份中的日期，范围为`1`到`31`，对于日期如`'0000-00-00'`或`'2008-00-00'`等具有零日期部分的日期，返回`0`。如果*`date`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT DAYOFMONTH('2007-02-03');
     -> 3
    ```

+   `DAYOFWEEK(*`date`*)`

    返回*`date`*的星期索引（`1` = 星期日，`2` = 星期一，...，`7` = 星期六）。这些索引值对应于 ODBC 标准。如果*`date`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT DAYOFWEEK('2007-02-03');
     -> 7
    ```

+   `DAYOFYEAR(*`date`*)`

    返回*`date`*的一年中的日期，范围为`1`到`366`。如果*`date`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT DAYOFYEAR('2007-02-03');
     -> 34
    ```

+   `EXTRACT(*`unit`* FROM *`date`*)`

    `EXTRACT()`函数使用与`DATE_ADD()`或`DATE_SUB()`相同类型的*`unit`*说明符，但是从日期中提取部分而不是执行日期算数。有关*`unit`*参数的信息，请参阅时间间隔。如果*`date`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT EXTRACT(YEAR FROM '2019-07-02');
     -> 2019
    mysql> SELECT EXTRACT(YEAR_MONTH FROM '2019-07-02 01:02:03');
     -> 201907
    mysql> SELECT EXTRACT(DAY_MINUTE FROM '2019-07-02 01:02:03');
     -> 20102
    mysql> SELECT EXTRACT(MICROSECOND
     ->                FROM '2003-01-02 10:30:00.000123');
     -> 123
    ```

+   `FROM_DAYS(*`N`*)`

    给定一个日期数*`N`*，返回一个`DATE`值。如果*`N`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT FROM_DAYS(730669);
     -> '2000-07-03'
    ```

    谨慎使用`FROM_DAYS()`处理旧日期。它不适用于格里高利历（1582 年之前）之前的值。请参阅第 13.2.7 节，“MySQL 使用的日历是什么？”。

+   [`FROM_UNIXTIME(*`unix_timestamp`*[,*`format`*])`](date-and-time-functions.html#function_from-unixtime)

    将 *`unix_timestamp`* 表示为日期时间或字符字符串值。返回的值使用会话时区表示。（客户端可以设置会话时区，如第 7.1.15 节，“MySQL 服务器时区支持”中所述。）*`unix_timestamp`* 是一个内部时间戳值，表示自 `'1970-01-01 00:00:00'` UTC 以来的秒数，例如`UNIX_TIMESTAMP()`函数生成的值。

    如果省略 *`format`*，此函数将返回一个`DATETIME`值。

    如果 *`unix_timestamp`* 或 *`format`* 为 `NULL`，此函数将返回 `NULL`。

    如果 *`unix_timestamp`* 是整数，则 `DATETIME` 的小数秒精度为零。当 *`unix_timestamp`* 是十进制值时，`DATETIME` 的小数秒精度与十进制值的精度相同，最多为 6。当 *`unix_timestamp`* 是浮点数时，日期时间的小数秒精度为 6。

    在 32 位平台上，*`unix_timestamp`* 的最大有用值为 2147483647.999999，返回 `'2038-01-19 03:14:07.999999'` UTC。在运行 MySQL 8.0.28 或更高版本的 64 位平台上，有效最大值为 32536771199.999999，返回 `'3001-01-18 23:59:59.999999'` UTC。无论平台或版本如何，*`unix_timestamp`* 的值大于有效最大值都将返回 `0`。

    *`format`* 用于以与`DATE_FORMAT()`函数使用的格式字符串相同的方式格式化结果。如果提供了 *`format`*，则返回的值是一个`VARCHAR`。

    ```sql
    mysql> SELECT FROM_UNIXTIME(1447430881);
     -> '2015-11-13 10:08:01'
    mysql> SELECT FROM_UNIXTIME(1447430881) + 0;
     -> 20151113100801
    mysql> SELECT FROM_UNIXTIME(1447430881,
     ->                      '%Y %D %M %h:%i:%s %x');
     -> '2015 13th November 10:08:01 2015'
    ```

    注意

    如果您使用`UNIX_TIMESTAMP()`和`FROM_UNIXTIME()`在非 UTC 时区和 Unix 时间戳值之间进行转换，转换是有损的，因为映射在两个方向上不是一对一的。有关详细信息，请参阅`UNIX_TIMESTAMP()`函数的描述。

+   `GET_FORMAT({DATE|TIME|DATETIME}, {'EUR'|'USA'|'JIS'|'ISO'|'INTERNAL'})`

    返回一个格式字符串。此函数与`DATE_FORMAT()`和`STR_TO_DATE()`函数结合使用时很有用。

    如果*`format`*为`NULL`，则此函数返回`NULL`。

    第一个和第二个参数的可能值会导致多个可能的格式字符串（有关使用的占位符，请参见`DATE_FORMAT()`函数描述中的表格）。ISO 格式指的是 ISO 9075，而不是 ISO 8601。

    | 函数调用 | 结果 |
    | --- | --- |
    | `GET_FORMAT(DATE,'USA')` | `'%m.%d.%Y'` |
    | `GET_FORMAT(DATE,'JIS')` | `'%Y-%m-%d'` |
    | `GET_FORMAT(DATE,'ISO')` | `'%Y-%m-%d'` |
    | `GET_FORMAT(DATE,'EUR')` | `'%d.%m.%Y'` |
    | `GET_FORMAT(DATE,'INTERNAL')` | `'%Y%m%d'` |
    | `GET_FORMAT(DATETIME,'USA')` | `'%Y-%m-%d %H.%i.%s'` |
    | `GET_FORMAT(DATETIME,'JIS')` | `'%Y-%m-%d %H:%i:%s'` |
    | `GET_FORMAT(DATETIME,'ISO')` | `'%Y-%m-%d %H:%i:%s'` |
    | `GET_FORMAT(DATETIME,'EUR')` | `'%Y-%m-%d %H.%i.%s'` |
    | `GET_FORMAT(DATETIME,'INTERNAL')` | `'%Y%m%d%H%i%s'` |
    | `GET_FORMAT(TIME,'USA')` | `'%h:%i:%s %p'` |
    | `GET_FORMAT(TIME,'JIS')` | `'%H:%i:%s'` |
    | `GET_FORMAT(TIME,'ISO')` | `'%H:%i:%s'` |
    | `GET_FORMAT(TIME,'EUR')` | `'%H.%i.%s'` |
    | `GET_FORMAT(TIME,'INTERNAL')` | `'%H%i%s'` |
    | 函数调用 | 结果 |

    `TIMESTAMP`也可以作为`GET_FORMAT()`的第一个参数使用，此时函数返回与`DATETIME`相同的值。

    ```sql
    mysql> SELECT DATE_FORMAT('2003-10-03',GET_FORMAT(DATE,'EUR'));
     -> '03.10.2003'
    mysql> SELECT STR_TO_DATE('10.31.2003',GET_FORMAT(DATE,'USA'));
     -> '2003-10-31'
    ```

+   `HOUR(*`time`*)`

    返回*`time`*的小时。对于一天中的时间值，返回值的范围是`0`到`23`。但是，`TIME`值的范围实际上要大得多，因此`HOUR`可能返回大于`23`的值。如果*`time`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT HOUR('10:05:03');
     -> 10
    mysql> SELECT HOUR('272:59:59');
     -> 272
    ```

+   `LAST_DAY(*`date`*)`

    获取一个日期或日期时间值，并返回该月的最后一天的相应值。如果参数无效或为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT LAST_DAY('2003-02-05');
     -> '2003-02-28'
    mysql> SELECT LAST_DAY('2004-02-05');
     -> '2004-02-29'
    mysql> SELECT LAST_DAY('2004-01-01 01:01:01');
     -> '2004-01-31'
    mysql> SELECT LAST_DAY('2003-03-32');
     -> NULL
    ```

+   `LOCALTIME`, [`LOCALTIME([*`fsp`*])`](date-and-time-functions.html#function_localtime)

    `LOCALTIME` 和 `LOCALTIME()` 是 `NOW()` 的同义词。

+   `LOCALTIMESTAMP`, [`LOCALTIMESTAMP([*`fsp`*])`](date-and-time-functions.html#function_localtimestamp)

    `LOCALTIMESTAMP` 和 `LOCALTIMESTAMP()` 是 `NOW()` 的同义词。

+   `MAKEDATE(*`year`*,*`dayofyear`*)`

    返回一个日期，给定年份和一年中的天数。*`dayofyear`* 必须大于 0，否则结果为`NULL`。如果任一参数为`NULL`，结果也为`NULL`。

    ```sql
    mysql> SELECT MAKEDATE(2011,31), MAKEDATE(2011,32);
     -> '2011-01-31', '2011-02-01'
    mysql> SELECT MAKEDATE(2011,365), MAKEDATE(2014,365);
     -> '2011-12-31', '2014-12-31'
    mysql> SELECT MAKEDATE(2011,0);
     -> NULL
    ```

+   `MAKETIME(*`hour`*,*`minute`*,*`second`*)`

    返回从*`hour`*、*`minute`*和*`second`*参数计算的时间值。如果任一参数为`NULL`，则返回`NULL`。

    *`second`* 参数可以有小数部分。

    ```sql
    mysql> SELECT MAKETIME(12,15,30);
     -> '12:15:30'
    ```

+   `MICROSECOND(*`expr`*)`

    返回从时间或日期时间表达式*`expr`*中计算的微秒，范围从`0`到`999999`的数字。如果*`expr`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT MICROSECOND('12:00:00.123456');
     -> 123456
    mysql> SELECT MICROSECOND('2019-12-31 23:59:59.000010');
     -> 10
    ```

+   `MINUTE(*`time`*)`

    返回*`time`*的分钟，范围为`0`到`59`，如果*`time`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT MINUTE('2008-02-03 10:05:03');
     -> 5
    ```

+   `MONTH(*`date`*)`

    返回*`date`*的月份，对于一月到十二月的范围为`1`到`12`，对于具有零月部分的日期（如`'0000-00-00'`或`'2008-00-00'`）为`0`。如果*`date`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT MONTH('2008-02-03');
     -> 2
    ```

+   `MONTHNAME(*`date`*)`

    返回*`date`*的月份的全名。名称的语言由`lc_time_names`系统变量的值控制（第 12.16 节，“MySQL 服务器区域设置支持”）。如果*`date`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT MONTHNAME('2008-02-03');
     -> 'February'
    ```

+   [`NOW([*`fsp`*])`](date-and-time-functions.html#function_now)

    返回当前日期和时间作为一个值，格式为`'*`YYYY-MM-DD hh:mm:ss`*'`或*`YYYYMMDDhhmmss`*，取决于函数在字符串或数字上下文中的使用。该值以会话时区表示。

    如果给定*`fsp`*参数以指定从 0 到 6 的小数秒精度，则返回值包括相应数量的小数秒部分。

    ```sql
    mysql> SELECT NOW();
     -> '2007-12-15 23:50:26'
    mysql> SELECT NOW() + 0;
     -> 20071215235026.000000
    ```

    `NOW()`返回一个常量时间，表示语句开始执行的时间。（在存储函数或触发器中，`NOW()`返回函数或触发语句开始执行的时间。）这与`SYSDATE()`的行为不同，后者返回执行时的确切时间。

    ```sql
    mysql> SELECT NOW(), SLEEP(2), NOW();
    +---------------------+----------+---------------------+
    | NOW()               | SLEEP(2) | NOW()               |
    +---------------------+----------+---------------------+
    | 2006-04-12 13:47:36 |        0 | 2006-04-12 13:47:36 |
    +---------------------+----------+---------------------+

    mysql> SELECT SYSDATE(), SLEEP(2), SYSDATE();
    +---------------------+----------+---------------------+
    | SYSDATE()           | SLEEP(2) | SYSDATE()           |
    +---------------------+----------+---------------------+
    | 2006-04-12 13:47:44 |        0 | 2006-04-12 13:47:46 |
    +---------------------+----------+---------------------+
    ```

    此外，`SET TIMESTAMP`语句会影响由`NOW()`返回的值，但不会影响`SYSDATE()`的返回值。这意味着二进制日志中的时间戳设置不会影响对`SYSDATE()`的调用。将时间戳设置为非零值会导致每次后续调用`NOW()`都返回该值。将时间戳设置为零会取消此效果，使得`NOW()`再次返回当前日期和时间。

    有关这两个函数之间的差异的更多信息，请参阅`SYSDATE()`的描述。

+   `PERIOD_ADD(*`P`*,*`N`*)`

    将*`N`*个月添加到格式为*`YYMM`*或*`YYYYMM`*的期间*`P`*中。返回格式为*`YYYYMM`*的值。

    注意

    期间参数*`P`*不是日期值。

    如果*`P`*或*`N`*为`NULL`，则此函数返回`NULL`。

    ```sql
    mysql> SELECT PERIOD_ADD(200801,2);
     -> 200803
    ```

+   `PERIOD_DIFF(*`P1`*,*`P2`*)`

    返回期间*`P1`*和*`P2`*之间的月份数。*`P1`*和*`P2`*应该是格式为*`YYMM`*或*`YYYYMM`*的值。请注意，期间参数*`P1`*和*`P2`*不是日期值。

    如果*`P1`*或*`P2`*为`NULL`，则此函数返回`NULL`。

    ```sql
    mysql> SELECT PERIOD_DIFF(200802,200703);
     -> 11
    ```

+   `QUARTER(*`date`*)`

    返回*`date`*的年份季度，范围为`1`到`4`，如果*`date`*为`NULL`则返回`NULL`。

    ```sql
    mysql> SELECT QUARTER('2008-04-01');
     -> 2
    ```

+   `SECOND(*`time`*)`

    返回*`time`*的秒数，范围为`0`到`59`，如果*`time`*为`NULL`则返回`NULL`。

    ```sql
    mysql> SELECT SECOND('10:05:03');
     -> 3
    ```

+   `SEC_TO_TIME(*`seconds`*)`

    返回*`seconds`*参数转换为小时、分钟和秒的`TIME`值。结果的范围受限于`TIME`数据类型的范围。如果参数对应的值超出该范围，则会发出警告。

    如果*`seconds`*为`NULL`，则函数返回`NULL`。

    ```sql
    mysql> SELECT SEC_TO_TIME(2378);
     -> '00:39:38'
    mysql> SELECT SEC_TO_TIME(2378) + 0;
     -> 3938
    ```

+   `STR_TO_DATE(*`str`*,*`format`*)`

    这是`DATE_FORMAT()` 函数的反向操作。它接受一个字符串*`str`*和一个格式字符串*`format`*。如果格式字符串同时包含日期和时间部分，则`STR_TO_DATE()`返回一个`DATETIME` 值，如果字符串仅包含日期或时间部分，则返回一个`DATE` 或`TIME` 值。如果*`str`*或*`format`*为`NULL`，则函数返回`NULL`。如果从*`str`*中提取的日期、时间或日期时间值无法按照服务器遵循的规则解析，则`STR_TO_DATE()` 返回`NULL`并生成警告。

    服务器扫描*`str`*，尝试将*`format`*与其匹配。格式字符串可以包含文字字符和以`%`开头的格式说明符。*`format`*中的文字字符必须与*`str`*中的文字字符完全匹配。*`format`*中的格式说明符必须与*`str`*中的日期或时间部分匹配。有关可用于*`format`*中的说明符，请参见`DATE_FORMAT()` 函数说明。

    ```sql
    mysql> SELECT STR_TO_DATE('01,5,2013','%d,%m,%Y');
     -> '2013-05-01'
    mysql> SELECT STR_TO_DATE('May 1, 2013','%M %d,%Y');
     -> '2013-05-01'
    ```

    扫描从*`str`*的开头开始，如果发现*`format`*不匹配，则失败。*`str`*末尾的额外字符将被忽略。

    ```sql
    mysql> SELECT STR_TO_DATE('a09:30:17','a%h:%i:%s');
     -> '09:30:17'
    mysql> SELECT STR_TO_DATE('a09:30:17','%h:%i:%s');
     -> NULL
    mysql> SELECT STR_TO_DATE('09:30:17a','%h:%i:%s');
     -> '09:30:17'
    ```

    未指定的日期或时间部分的值为 0，因此在*`str`*中未完全指定的值将产生一个结果，其中一些或所有部分设置为 0：

    ```sql
    mysql> SELECT STR_TO_DATE('abc','abc');
     -> '0000-00-00'
    mysql> SELECT STR_TO_DATE('9','%m');
     -> '0000-09-00'
    mysql> SELECT STR_TO_DATE('9','%s');
     -> '00:00:09'
    ```

    日期值的部分的范围检查如第 13.2.2 节，“日期、日期时间和时间戳类型”中所述。这意味着，例如，“零”日期或部分值为 0 的日期是允许的，除非 SQL 模式设置为不允许这些值。

    ```sql
    mysql> SELECT STR_TO_DATE('00/00/0000', '%m/%d/%Y');
     -> '0000-00-00'
    mysql> SELECT STR_TO_DATE('04/31/2004', '%m/%d/%Y');
     -> '2004-04-31'
    ```

    如果启用了`NO_ZERO_DATE` SQL 模式，则不允许零日期。在这种情况下，`STR_TO_DATE()` 返回`NULL`并生成警告：

    ```sql
    mysql> SET sql_mode = '';
    mysql> SELECT STR_TO_DATE('00/00/0000', '%m/%d/%Y');
    +---------------------------------------+
    | STR_TO_DATE('00/00/0000', '%m/%d/%Y') |
    +---------------------------------------+
    | 0000-00-00                            |
    +---------------------------------------+
    mysql> SET sql_mode = 'NO_ZERO_DATE';
    mysql> SELECT STR_TO_DATE('00/00/0000', '%m/%d/%Y');
    +---------------------------------------+
    | STR_TO_DATE('00/00/0000', '%m/%d/%Y') |
    +---------------------------------------+
    | NULL                                  |
    +---------------------------------------+
    mysql> SHOW WARNINGS\G
    *************************** 1\. row ***************************
      Level: Warning
       Code: 1411
    Message: Incorrect datetime value: '00/00/0000' for function str_to_date
    ```

    在 MySQL 8.0.35 之前，可以将无效的日期字符串（例如 `'2021-11-31'`）传递给此函数。在 MySQL 8.0.35 及更高版本中，`STR_TO_DATE()` 执行完整的范围检查，并在转换后的日期无效时引发错误。

    注意

    你不能使用格式`"%X%V"`将年周字符串转换为日期，因为如果周跨越月边界，则年和周的组合不能唯一标识年和月。要将年周转换为日期，还应指定星期几：

    ```sql
    mysql> SELECT STR_TO_DATE('200442 Monday', '%X%V %W');
     -> '2004-10-18'
    ```

    您还应该注意，对于日期和日期时间值的日期部分，`STR_TO_DATE()`仅检查年、月和日的有效性。更准确地说，这意味着检查年份以确保它在 0-9999 的范围内，检查月份以确保它在 1-12 的范围内，检查日期以确保它在 1-31 的范围内，但服务器不会检查这些值的组合。例如，`SELECT STR_TO_DATE('23-2-31', '%Y-%m-%d')`返回`2023-02-31`。启用或禁用`ALLOW_INVALID_DATES`服务器 SQL 模式对此行为没有影响。有关更多信息，请参阅第 13.2.2 节，“DATE、DATETIME 和 TIMESTAMP 类型”。

+   `SUBDATE(*`date`*,INTERVAL *`expr`* *`unit`*)`，`SUBDATE(*`expr`*,*`days`*)`

    当使用第二个参数的`INTERVAL`形式调用时，`SUBDATE()`是`DATE_SUB()`的同义词。有关`INTERVAL` *`unit`*参数的信息，请参阅`DATE_ADD()`的讨论。

    ```sql
    mysql> SELECT DATE_SUB('2008-01-02', INTERVAL 31 DAY);
     -> '2007-12-02'
    mysql> SELECT SUBDATE('2008-01-02', INTERVAL 31 DAY);
     -> '2007-12-02'
    ```

    第二种形式允许使用整数值作为*`days`*。在这种情况下，它被解释为要从日期或日期时间表达式*`expr`*中减去的天数。

    ```sql
    mysql> SELECT SUBDATE('2008-01-02 12:00:00', 31);
     -> '2007-12-02 12:00:00'
    ```

    如果任何参数为`NULL`，则此函数返回`NULL`。

+   `SUBTIME(*`expr1`*,*`expr2`*)`

    `SUBTIME()`返回*`expr1`* − *`expr2`*，以与*`expr1`*相同格式的值表示。*`expr1`*是时间或日期时间表达式，*`expr2`*是时间表达式。

    此函数返回类型的分辨率与`ADDTIME()`函数的执行方式相同；有关更多信息，请参阅该函数的描述。

    ```sql
    mysql> SELECT SUBTIME('2007-12-31 23:59:59.999999','1 1:1:1.000002');
     -> '2007-12-30 22:58:58.999997'
    mysql> SELECT SUBTIME('01:00:00.999999', '02:00:00.999998');
     -> '-00:59:59.999999'
    ```

    如果*`expr1`*或*`expr2`*为`NULL`，则此函数返回`NULL`。

+   [`SYSDATE([*`fsp`*])`](date-and-time-functions.html#function_sysdate)

    返回当前日期和时间作为值，格式为`'*`YYYY-MM-DD hh:mm:ss`*'`或*`YYYYMMDDhhmmss`*，具体取决于函数在字符串或数字上下文中的使用方式。

    如果给定*`fsp`*参数以指定从 0 到 6 的小数秒精度，则返回值包括该数量的小数秒部分。

    `SYSDATE()`返回其执行时的时间。这与`NOW()`的行为不同，后者返回指示语句开始执行的时间的常量时间。（在存储函数或触发器中，`NOW()`返回函数或触发语句开始执行的时间。）

    ```sql
    mysql> SELECT NOW(), SLEEP(2), NOW();
    +---------------------+----------+---------------------+
    | NOW()               | SLEEP(2) | NOW()               |
    +---------------------+----------+---------------------+
    | 2006-04-12 13:47:36 |        0 | 2006-04-12 13:47:36 |
    +---------------------+----------+---------------------+

    mysql> SELECT SYSDATE(), SLEEP(2), SYSDATE();
    +---------------------+----------+---------------------+
    | SYSDATE()           | SLEEP(2) | SYSDATE()           |
    +---------------------+----------+---------------------+
    | 2006-04-12 13:47:44 |        0 | 2006-04-12 13:47:46 |
    +---------------------+----------+---------------------+
    ```

    此外，`SET TIMESTAMP`语句会影响`NOW()`返回的值，但不会影响`SYSDATE()`返回的值。这意味着二进制日志中的时间戳设置对`SYSDATE()`的调用没有影响。

    因为`SYSDATE()`甚至在同一语句中可能返回不同的值，并且不受`SET TIMESTAMP`的影响，因此它是不确定的，因此在使用基于语句的二进制日志记录时不安全。如果这是一个问题，您可以使用基于行的日志记录。

    或者，您可以使用`--sysdate-is-now`选项，使`SYSDATE()`成为`NOW()`的别名。如果在复制源服务器和副本上都使用该选项，则有效。

    `SYSDATE()`的不确定性特性也意味着无法使用索引来评估引用它的表达式。

+   `TIME(*`expr`*)`

    提取时间或日期时间表达式*`expr`*的时间部分，并将其作为字符串返回。如果*`expr`*为`NULL`，则返回`NULL`。

    此函数对基于语句的复制不安全。如果在`binlog_format`设置为`STATEMENT`时使用此函数，将记录警告。

    ```sql
    mysql> SELECT TIME('2003-12-31 01:02:03');
     -> '01:02:03'
    mysql> SELECT TIME('2003-12-31 01:02:03.000123');
     -> '01:02:03.000123'
    ```

+   `TIMEDIFF(*`expr1`*,*`expr2`*)`

    `TIMEDIFF()`将*`expr1`* − *`expr2`*表示为时间值。*`expr1`*和*`expr2`*是转换为`TIME`或`DATETIME`表达式的字符串；在转换后，它们必须是相同类型的。如果*`expr1`*或*`expr2`*为`NULL`，则返回`NULL`。

    `TIMEDIFF()`返回的结果受限于允许的`TIME`值的范围。或者，您可以使用`TIMESTAMPDIFF()`和`UNIX_TIMESTAMP()`中的任一函数，两者都返回整数。

    ```sql
    mysql> SELECT TIMEDIFF('2000-01-01 00:00:00',
     ->                 '2000-01-01 00:00:00.000001');
     -> '-00:00:00.000001'
    mysql> SELECT TIMEDIFF('2008-12-31 23:59:59.000001',
     ->                 '2008-12-30 01:01:01.000002');
     -> '46:58:57.999999'
    ```

+   `TIMESTAMP(*`expr`*)`, `TIMESTAMP(*`expr1`*,*`expr2`*)`

    使用单个参数时，该函数将日期或日期时间表达式*`expr`*作为日期时间值返回。使用两个参数时，它将时间表达式*`expr2`*添加到日期或日期时间表达式*`expr1`*中，并将结果作为日期时间值返回。如果*`expr`*、*`expr1`*或*`expr2`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT TIMESTAMP('2003-12-31');
     -> '2003-12-31 00:00:00'
    mysql> SELECT TIMESTAMP('2003-12-31 12:00:00','12:00:00');
     -> '2004-01-01 00:00:00'
    ```

+   `TIMESTAMPADD(*`unit`*,*`interval`*,*`datetime_expr`*)`

    将整数表达式*`interval`*添加到日期或日期时间表达式*`datetime_expr`*中。*`interval`*的单位由*`unit`*参数给出，应为以下值之一：`MICROSECOND`（微秒）、`SECOND`、`MINUTE`、`HOUR`、`DAY`、`WEEK`、`MONTH`、`QUARTER`或`YEAR`。

    *`unit`*值可以使用如下所示的关键字之一指定，也可以使用`SQL_TSI_`前缀。例如，`DAY`和`SQL_TSI_DAY`都是合法的。

    如果*`interval`*或*`datetime_expr`*为`NULL`，则此函数返回`NULL`。

    ```sql
    mysql> SELECT TIMESTAMPADD(MINUTE, 1, '2003-01-02');
     -> '2003-01-02 00:01:00'
    mysql> SELECT TIMESTAMPADD(WEEK,1,'2003-01-02');
     -> '2003-01-09'
    ```

    当向`DATE`或`DATETIME`值添加`MONTH`间隔时，如果结果日期包含给定月份中不存在的日期，则将日期调整为该月的最后一天，如下所示：

    ```sql
    mysql> SELECT TIMESTAMPADD(MONTH, 1, DATE '2024-03-30') AS t1, 
         >        TIMESTAMPADD(MONTH, 1, DATE '2024-03-31') AS t2;
    +------------+------------+
    | t1         | t2         |
    +------------+------------+
    | 2024-04-30 | 2024-04-30 |
    +------------+------------+
    1 row in set (0.00 sec)
    ```

+   `TIMESTAMPDIFF(*`unit`*,*`datetime_expr1`*,*`datetime_expr2`*)`

    返回*`datetime_expr2`* − *`datetime_expr1`*，其中*`datetime_expr1`*和*`datetime_expr2`*是日期或日期时间表达式。一个表达式可以是日期，另一个可以是日期时间；日期值在必要时被视为具有时间部分`'00:00:00'`的日期时间。结果（整数）的单位由*`unit`*参数给出。*`unit`*的合法值与`TIMESTAMPADD()`函数的描述中列出的相同。

    如果*`datetime_expr1`*或*`datetime_expr2`*为`NULL`，则此函数返回`NULL`。

    ```sql
    mysql> SELECT TIMESTAMPDIFF(MONTH,'2003-02-01','2003-05-01');
     -> 3
    mysql> SELECT TIMESTAMPDIFF(YEAR,'2002-05-01','2001-01-01');
     -> -1
    mysql> SELECT TIMESTAMPDIFF(MINUTE,'2003-02-01','2003-05-01 12:05:55');
     -> 128885
    ```

    注意

    该函数的日期或日期时间参数的顺序与使用 2 个参数调用`TIMESTAMP()`函数时相反。

+   `TIME_FORMAT(*`time`*,*`format`*)`

    这类似于`DATE_FORMAT()`函数，但*`format`*字符串可能仅包含有关小时、分钟、秒和微秒的格式说明符。其他说明符会产生`NULL`或`0`。如果*`time`*或*`format`*为`NULL`，则`TIME_FORMAT()`返回`NULL`。

    如果*`time`*值包含大于`23`的小时部分，则`%H`和`%k`小时格式说明符会产生大于通常范围的`0..23`的值。其他小时格式说明符会将小时值对`12`取模。

    ```sql
    mysql> SELECT TIME_FORMAT('100:00:00', '%H %k %h %I %l');
     -> '100 100 04 04 4'
    ```

+   `TIME_TO_SEC(*`time`*)`

    返回将*`time`*参数转换为秒的结果。如果*`time`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT TIME_TO_SEC('22:23:00');
     -> 80580
    mysql> SELECT TIME_TO_SEC('00:39:38');
     -> 2378
    ```

+   `TO_DAYS(*`date`*)`

    给定一个日期*`date`*，返回一个日期编号（自公元 0 年以来的天数）。如果*`date`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT TO_DAYS(950501);
     -> 728779
    mysql> SELECT TO_DAYS('2007-10-07');
     -> 733321
    ```

    `TO_DAYS()`不适用于格里高利历（1582 年）出现之前的值，因为它没有考虑到在日历更改时丢失的天数。对于 1582 年之前的日期（可能是其他地区的较晚年份），此函数的结果不可靠。有关详细信息，请参阅第 13.2.7 节，“MySQL 使用的日历是什么？”。

    请记住，MySQL 将日期中的两位年份值转换为四位形式，使用的规则在第 13.2 节，“日期和时间数据类型”中。例如，`'2008-10-07'`和`'08-10-07'`被视为相同的日期：

    ```sql
    mysql> SELECT TO_DAYS('2008-10-07'), TO_DAYS('08-10-07');
     -> 733687, 733687
    ```

    在 MySQL 中，零日期被定义为`'0000-00-00'`，即使这个日期本身被认为是无效的。这意味着，对于`'0000-00-00'`和`'0000-01-01'`，`TO_DAYS()`返回以下值：

    ```sql
    mysql> SELECT TO_DAYS('0000-00-00');
    +-----------------------+
    | to_days('0000-00-00') |
    +-----------------------+
    |                  NULL |
    +-----------------------+
    1 row in set, 1 warning (0.00 sec)

    mysql> SHOW WARNINGS;
    +---------+------+----------------------------------------+
    | Level   | Code | Message                                |
    +---------+------+----------------------------------------+
    | Warning | 1292 | Incorrect datetime value: '0000-00-00' |
    +---------+------+----------------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT TO_DAYS('0000-01-01');
    +-----------------------+
    | to_days('0000-01-01') |
    +-----------------------+
    |                     1 |
    +-----------------------+
    1 row in set (0.00 sec)
    ```

    无论是否启用`ALLOW_INVALID_DATES` SQL 服务器模式，这都是正确的。

+   `TO_SECONDS(*`expr`*)`

    给定一个日期或日期时间*`expr`*，返回自公元 0 年以来的秒数。如果*`expr`*不是有效的日期或日期时间值（包括`NULL`），则返回`NULL`。

    ```sql
    mysql> SELECT TO_SECONDS(950501);
     -> 62966505600
    mysql> SELECT TO_SECONDS('2009-11-29');
     -> 63426672000
    mysql> SELECT TO_SECONDS('2009-11-29 13:43:32');
     -> 63426721412
    mysql> SELECT TO_SECONDS( NOW() );
     -> 63426721458
    ```

    像`TO_DAYS()`一样，`TO_SECONDS()`不适用于格里高利历（1582 年）出现之前的值，因为它没有考虑到在日历更改时丢失的天数。对于 1582 年之前的日期（可能是其他地区的较晚年份），此函数的结果不可靠。有关详细信息，请参阅第 13.2.7 节，“MySQL 使用的日历是什么？”。

    像`TO_DAYS()`一样，`TO_SECONDS()`，将日期中的两位年份值转换为四位形式，使用的规则在第 13.2 节，“日期和时间数据类型”中。

    在 MySQL 中，零日期被定义为`'0000-00-00'`，即使这个日期本身被认为是无效的。这意味着，对于`'0000-00-00'`和`'0000-01-01'`，`TO_SECONDS()`返回以下值：

    ```sql
    mysql> SELECT TO_SECONDS('0000-00-00');
    +--------------------------+
    | TO_SECONDS('0000-00-00') |
    +--------------------------+
    |                     NULL |
    +--------------------------+
    1 row in set, 1 warning (0.00 sec)

    mysql> SHOW WARNINGS;
    +---------+------+----------------------------------------+
    | Level   | Code | Message                                |
    +---------+------+----------------------------------------+
    | Warning | 1292 | Incorrect datetime value: '0000-00-00' |
    +---------+------+----------------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT TO_SECONDS('0000-01-01');
    +--------------------------+
    | TO_SECONDS('0000-01-01') |
    +--------------------------+
    |                    86400 |
    +--------------------------+
    1 row in set (0.00 sec)
    ```

    无论是否启用`ALLOW_INVALID_DATES` SQL 服务器模式，这都是正确的。

+   [`UNIX_TIMESTAMP([*`date`*])`](date-and-time-functions.html#function_unix-timestamp)

    如果调用`UNIX_TIMESTAMP()`时没有*`date`*参数，它将返回一个表示自`'1970-01-01 00:00:00'` UTC 以来的秒数的 Unix 时间戳。

    如果使用*`date`*参数调用`UNIX_TIMESTAMP()`，它将返回自`'1970-01-01 00:00:00'` UTC 以来的秒数值。服务器将*`date`*解释为会话时区中的值，并将其转换为 UTC 中的内部 Unix 时间戳值。（客户端可以根据第 7.1.15 节“MySQL 服务器时区支持”中的描述设置会话时区。）*`date`*参数可以是`DATE`、`DATETIME`或`TIMESTAMP`字符串，或以*`YYMMDD`*、*`YYMMDDhhmmss`*、*`YYYYMMDD`*或*`YYYYMMDDhhmmss`*格式的数字。如果参数包括时间部分，则可以选择包括小数秒部分。

    如果没有给定参数或参数不包括小数秒部分，则返回值为整数，或者给定包括小数秒部分的参数，则返回`DECIMAL`。

    当*`date`*参数是`TIMESTAMP`列时，`UNIX_TIMESTAMP()`直接返回内部时间戳值，没有隐式的“字符串到 Unix 时间戳”的转换。

    在 MySQL 8.0.28 之前，参数值的有效范围与`TIMESTAMP`数据类型相同：`'1970-01-01 00:00:01.000000'` UTC 到`'2038-01-19 03:14:07.999999'` UTC。对于运行在 64 位平台上的 MySQL 8.0.28 及更高版本，`UNIX_TIMESTAMP()`的参数值的有效范围为`'1970-01-01 00:00:01.000000'` UTC 到`'3001-01-19 03:14:07.999999'` UTC（对应 32536771199.999999 秒）。

    无论 MySQL 版本或平台架构如何，如果将超出范围的日期传递给`UNIX_TIMESTAMP()`，它将返回`0`。如果*`date`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT UNIX_TIMESTAMP();
     -> 1447431666
    mysql> SELECT UNIX_TIMESTAMP('2015-11-13 10:20:19');
     -> 1447431619
    mysql> SELECT UNIX_TIMESTAMP('2015-11-13 10:20:19.012');
     -> 1447431619.012
    ```

    如果使用`UNIX_TIMESTAMP()`和`FROM_UNIXTIME()`在非协调世界时时区和 Unix 时间戳值之间进行转换，转换是有损的，因为映射在两个方向上不是一对一的。例如，由于夏令时等本地时区更改的惯例，可能导致`UNIX_TIMESTAMP()`将两个在非协调世界时时区中不同的值映射到相同的 Unix 时间戳值。`FROM_UNIXTIME()`将该值映射回原始值中的一个。以下是一个示例，使用在`MET`时区中不同的值：

    ```sql
    mysql> SET time_zone = 'MET';
    mysql> SELECT UNIX_TIMESTAMP('2005-03-27 03:00:00');
    +---------------------------------------+
    | UNIX_TIMESTAMP('2005-03-27 03:00:00') |
    +---------------------------------------+
    |                            1111885200 |
    +---------------------------------------+
    mysql> SELECT UNIX_TIMESTAMP('2005-03-27 02:00:00');
    +---------------------------------------+
    | UNIX_TIMESTAMP('2005-03-27 02:00:00') |
    +---------------------------------------+
    |                            1111885200 |
    +---------------------------------------+
    mysql> SELECT FROM_UNIXTIME(1111885200);
    +---------------------------+
    | FROM_UNIXTIME(1111885200) |
    +---------------------------+
    | 2005-03-27 03:00:00       |
    +---------------------------+
    ```

    注意

    要使用诸如`'MET'`或`'Europe/Amsterdam'`之类的命名时区，必须正确设置时区表。有关说明，请参见第 7.1.15 节，“MySQL 服务器时区支持”。

    如果要减去`UNIX_TIMESTAMP()`列，可能需要将它们转换为有符号整数。参见第 14.10 节，“转换函数和运算符”。

+   `UTC_DATE`，`UTC_DATE()`

    返回当前的协调世界时日期作为一个值，格式为`'*`YYYY-MM-DD`*'`或*`YYYYMMDD`*，取决于函数在字符串或数字上下文中的使用方式。

    ```sql
    mysql> SELECT UTC_DATE(), UTC_DATE() + 0;
     -> '2003-08-14', 20030814
    ```

+   `UTC_TIME`，[`UTC_TIME([*`fsp`*])`](date-and-time-functions.html#function_utc-time)

    返回当前的协调世界时时间作为一个值，格式为*`'hh:mm:ss'`*或*`hhmmss`*，取决于函数在字符串或数字上下文中的使用方式。

    如果给定*`fsp`*参数以指定从 0 到 6 的小数秒精度，则返回值包括相应数量的小数秒部分。

    ```sql
    mysql> SELECT UTC_TIME(), UTC_TIME() + 0;
     -> '18:07:53', 180753.000000
    ```

+   `UTC_TIMESTAMP`，[`UTC_TIMESTAMP([*`fsp`*])`](date-and-time-functions.html#function_utc-timestamp)

    返回当前的协调世界时日期和时间作为一个值，格式为`'*`YYYY-MM-DD hh:mm:ss`*'`或*`YYYYMMDDhhmmss`*，取决于函数在字符串或数字上下文中的使用方式。

    如果给定*`fsp`*参数以指定从 0 到 6 的小数秒精度，则返回值包括相应数量的小数秒部分。

    ```sql
    mysql> SELECT UTC_TIMESTAMP(), UTC_TIMESTAMP() + 0;
     -> '2003-08-14 18:08:04', 20030814180804.000000
    ```

+   [`WEEK(*`date`*[,*`mode`*])`](date-and-time-functions.html#function_week)

    此函数返回*`date`*的周数。`WEEK()`的两参数形式使您能够指定周从星期日或星期一开始，返回值应该在`0`到`53`或`1`到`53`的范围内。如果省略*`mode`*参数，则使用`default_week_format`系统变量的值。请参见第 7.1.8 节，“服务器系统变量”。对于`NULL`日期值，函数返回`NULL`。

    以下表描述了*`mode`*参数的工作方式。

    | 模式 | 一周的第一天 | 范围 | 第 1 周是第一周 … |
    | --- | --- | --- | --- |
    | 0 | 星期日 | 0-53 | 今年有一个星期日 |
    | 1 | 星期一 | 0-53 | 今年有 4 天或更多天 |
    | 2 | 星期日 | 1-53 | 今年有一个星期日 |
    | 3 | 星期一 | 1-53 | 今年有 4 天或更多天 |
    | 4 | 星期日 | 0-53 | 今年有 4 天或更多天 |
    | 5 | 星期一 | 0-53 | 今年有一个星期一 |
    | 6 | 星期日 | 1-53 | 今年有 4 天或更多天 |
    | 7 | 星期一 | 1-53 | 今年有一个星期一 |

    对于具有“今年有 4 天或更多天”的*`mode`*值，周数按照 ISO 8601:1988 编号：

    +   如果包含 1 月 1 日的那一周有 4 天或更多天，那么它就是第 1 周。

    +   否则，它就是上一年的最后一周，下一周就是第 1 周。

    ```sql
    mysql> SELECT WEEK('2008-02-20');
     -> 7
    mysql> SELECT WEEK('2008-02-20',0);
     -> 7
    mysql> SELECT WEEK('2008-02-20',1);
     -> 8
    mysql> SELECT WEEK('2008-12-31',1);
     -> 53
    ```

    如果一个日期落在上一年的最后一周，且您没有使用`2`、`3`、`6`或`7`作为可选的*`mode`*参数，则 MySQL 将返回`0`：

    ```sql
    mysql> SELECT YEAR('2000-01-01'), WEEK('2000-01-01',0);
     -> 2000, 0
    ```

    有人可能会认为`WEEK()`应该返回`52`，因为给定的日期实际上出现在 1999 年的第 52 周。但`WEEK()`返回`0`，以便返回值是“给定年份中的周数”。这使得当与从日期中提取日期部分的其他函数结合使用时，`WEEK()`函数是可靠的。

    如果您希望结果相对于包含给定日期的一周的第一天的年份进行评估，请使用`0`、`2`、`5`或`7`作为可选的*`mode`*参数。

    ```sql
    mysql> SELECT WEEK('2000-01-01',2);
     -> 52
    ```

    或者，使用`YEARWEEK()`函数：

    ```sql
    mysql> SELECT YEARWEEK('2000-01-01');
     -> 199952
    mysql> SELECT MID(YEARWEEK('2000-01-01'),5,2);
     -> '52'
    ```

+   `WEEKDAY(*`date`*)`

    返回*`date`*的星期索引（`0` = 星期一，`1` = 星期二，… `6` = 星期日）。如果*`date`*为`NULL`，则返回`NULL`。

    ```sql
    mysql> SELECT WEEKDAY('2008-02-03 22:23:00');
     -> 6
    mysql> SELECT WEEKDAY('2007-11-06');
     -> 1
    ```

+   `WEEKOFYEAR(*`date`*)`

    返回日期的日历周作为范围从`1`到`53`的数字。如果*`date`*为`NULL`，则返回`NULL`。

    `WEEKOFYEAR()`是一个兼容函数，等效于`WEEK(*`date`*,3)`。

    ```sql
    mysql> SELECT WEEKOFYEAR('2008-02-20');
     -> 8
    ```

+   `YEAR(*`date`*)`

    返回 *`date`* 的年份，范围为 `1000` 到 `9999`，或者对于“零”日期为 `0`。如果 *`date`* 为 `NULL`，则返回 `NULL`。

    ```sql
    mysql> SELECT YEAR('1987-01-01');
     -> 1987
    ```

+   `YEARWEEK(*`date`*)`, `YEARWEEK(*`date`*,*`mode`*)`

    返回日期的年份和周数。结果中的年份可能与年份参数中的年份在一年中的第一周和最后一周不同。如果 *`date`* 为 `NULL`，则返回 `NULL`。

    *`mode`* 参数的工作方式与 `WEEK()` 函数的 *`mode`* 参数完全相同。对于单参数语法，使用 *`mode`* 值为 0。与 `WEEK()` 不同，`default_week_format` 的值不会影响 `YEARWEEK()`。

    ```sql
    mysql> SELECT YEARWEEK('1987-01-01');
     -> 198652
    ```

    周数与 `WEEK()` 函数对可选参数 `0` 或 `1` 返回的周数 (`0`) 不同，因为 `WEEK()` 然后返回给定年份上下文中的周数。
