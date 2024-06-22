# 1\. 概述

> 原文：[`sqlite.com/lang_datefunc.html`](https://sqlite.com/lang_datefunc.html)

SQLite 支持七种标量日期和时间函数，如下所示：

1.  **date(***time-value, modifier, modifier, ...***)**

1.  **time(***time-value, modifier, modifier, ...***)**

1.  **datetime(***time-value, modifier, modifier, ...***)**

1.  **julianday(***time-value, modifier, modifier, ...***)**

1.  **unixepoch(***time-value, modifier, modifier, ...***)**

1.  **strftime(***format, time-value, modifier, modifier, ...***)**

1.  **timediff(***time-value, time-value***)**

前六个日期和时间函数以可选的 time-value 作为参数，后面跟零个或多个 modifiers。strftime()函数的第一个参数也是格式字符串。timediff()函数接受两个参数，这两个参数都是 time-values。

SQLite 没有专门的日期/时间数据类型。相反，日期和时间值可以存储为以下任一格式之一：

> | [ISO-8601](http://zh.wikipedia.org/wiki/ISO_8601) | 一个文本字符串，表示 ISO 8601 日期/时间值。例如：`'2025-05-29 14:16:00'` |
> | --- | --- |
> | [朱利安日数](http://zh.wikipedia.org/wiki/朱利安日) | 自公元前 4713 年 11 月 24 日 12:00:00 以来的天数，包括小数天数。例如：`2460825.09444444` |
> | [Unix 时间戳](https://zh.wikipedia.org/wiki/Unix 时间) | 自 1970-01-01 00:00:00 以来的秒数，包括小数秒数。例如：`1748528160` |

这三种格式统称为 time-values。所有日期时间函数都接受 ISO-8601 文本或朱利安日数作为 time-values。它们也可以通过添加可选的 modifiers 参数'auto'或'unixepoch'来接受 Unix 时间戳。由于 timediff()函数不接受 modifiers，因此它只能使用 ISO-8601 和朱利安日数的 time-values。

**date()** 函数以 YYYY-MM-DD 格式返回文本形式的日期。

**time()** 函数以 HH:MM:SS 或如果使用 subsec modifier，则以 HH:MM:SS.SSS 格式返回文本形式的时间。

**datetime()** 函数以 YYYY-MM-DD HH:MM:SS 或如果使用 subsec modifier，则以 YYYY-MM-DD HH:MM:SS.SSS 格式返回日期和时间。

**julianday()** 函数返回[朱利安日](http://zh.wikipedia.org/wiki/朱利安日) - 自公元前 4714 年 11 月 24 日格林威治的中午起的天数，采用[普罗莱普特·格里高利历](http://zh.wikipedia.org/wiki/普罗莱普特·格里高利历)。

**unixepoch()** 函数返回 Unix 时间戳 - 自 1970-01-01 00:00:00 UTC 以来的秒数。unixepoch()函数通常返回整数秒数，但使用可选的 subsec modifier 时，将返回浮点数，即秒的小数部分。

**strftime()**函数根据作为第一个参数指定的格式字符串返回格式化的日期。格式字符串支持标准 C 库中 strftime()函数中找到的最常见的替换，加上两个新的替换，%f 和%J。以下是截至版本 3.46.0（2024-05-23）的有效 strftime()替换的完整列表。SQLite 的早期版本可能不支持所有替换。如果看到未定义或不支持的替换，则结果为 NULL。

> |  |  |  |
> | --- | --- | --- |
> | %d |  | 月份中的日期：01-31 |
> | %e |  | 没有前导零的月份中的日期：1-31 |
> | %f |  | 分数秒：SS.SSS |
> | %F |  | ISO 8601 日期：YYYY-MM-DD |
> | %G |  | 对应于%V 的 ISO 8601 年份 |
> | %g |  | 对应于%V 的两位数 ISO 8601 年份 |
> | %H |  | 小时：00-24 |
> | %I |  | 12 小时制小时：01-12 |
> | %j |  | 年份中的天数：001-366 |
> | %J |  | 朱利安日数（小数） |
> | %k |  | 没有前导零的小时：0-24 |
> | %l |  | 没有前导零的%I：1-12 |
> | %m |  | 月份：01-12 |
> | %M |  | 分钟：00-59 |
> | %p |  | 根据小时为“AM”或“PM” |
> | %P |  | 根据小时为“am”或“pm” |
> | %R |  | ISO 8601 时间：HH:MM |
> | %s |  | 自 1970-01-01 以来的秒数 |
> | %S |  | 秒数：00-59 |
> | %T |  | ISO 8601 时间：HH:MM:SS |
> | %U |  | 年份中的周数（00-53）- 第一个星期从周日开始 |
> | %u |  | 一周中的星期几，星期一为 1 |
> | %V |  | ISO 8601 年份中的周数 |
> | %w |  | 一周中的星期几，星期日为 0 |
> | %W |  | 年份中的周数（00-53）- 第一个星期从周一开始 |
> | %Y |  | 年份：0000-9999 |
> | %% |  | % |

其他日期和时间函数可以通过 strftime()来表达：

> | **Function** |  | **Equivalent strftime()** |
> | --- | --- | --- |
> | date(...) |  | strftime('%F', ...) |
> | time(...) |  | strftime('%T', ...) |
> | datetime(...) |  | strftime('%F %T', ...) |
> | julianday(...) |  | <nobr>CAST(strftime('%J', ...) as REAL)</nobr> |
> | unixepoch(...) |  | <nobr>CAST(strftime('%s', ...) as INT)</nobr> |

date(), time()和 datetime()函数都返回文本，因此它们的 strftime()等效值是准确的。然而，julianday()和 unixepoch()函数返回数值。它们的 strftime()等效值返回的是相应数字的文本表示。

为了方便和效率考虑，提供了除了 strftime()之外的函数。julianday()和 unixepoch()函数分别返回实数和整数值，并且不会产生与使用'%J'或'%s'格式说明符时 strftime()函数所产生的格式转换成本或不精确性相关的开销。

**timediff(A,B)**函数返回描述必须添加到 B 以达到时间 A 的时间量的字符串。timediff()结果的格式设计为人类可读。格式如下：

> (+|-)YYYY-MM-DD HH:MM:SS.SSS

这个时间差字符串也是其他日期/时间函数的允许修饰符。时间值 A 和 B 的以下不变性成立：

> datetime(A) = datetime(B, timediff(A,B))

月份和年份的长度各不相同。二月比三月短。闰年比非闰年长。timediff() 的输出考虑到了这一切。timediff() 函数旨在提供时间跨度的用户友好描述。如果你想知道两个日期 A 和 B 之间的天数或秒数，则可以始终执行以下操作之一：

> SELECT julianday(B) - julianday(A);
> 
> SELECT unixepoch(B) - unixepoch(A);

即使 A 和 B 的值跨越不同天数，timediff(A,B) 可能会返回相同的结果 - 取决于起始日期。例如，以下两个 timediff() 调用返回相同的结果 ("-0000-01-00 00:00:00.000")，尽管第一个时间跨度为 28 天，第二个为 31 天：

> SELECT timediff('2023-02-15','2023-03-15');
> 
> SELECT timediff('2023-03-15','2023-04-15');

摘要：如果你想要一个用户友好的时间跨度，请使用 timediff()。如果你需要精确的时间差（以天或秒计算），请使用 julianday() 或 unixepoch() 调用的差异。

# 2\. 时间值

时间值可以采用下面显示的任何格式。值通常是一个字符串，但在格式 12 的情况下可以是整数或浮点数。

1.  *YYYY-MM-DD*

1.  *YYYY-MM-DD HH:MM*

1.  *YYYY-MM-DD HH:MM:SS*

1.  *YYYY-MM-DD HH:MM:SS.SSS*

1.  *YYYY-MM-DD***T***HH:MM*

1.  *YYYY-MM-DD***T***HH:MM:SS*

1.  *YYYY-MM-DD***T***HH:MM:SS.SSS*

1.  *HH:MM*

1.  *HH:MM:SS*

1.  *HH:MM:SS.SSS*

1.  **now**

1.  *DDDDDDDDDD*

在格式 5 到 7 中，“T” 是一个文字字符，用于分隔日期和时间，符合 [ISO-8601](http://www.w3c.org/TR/NOTE-datetime) 的要求。格式 8 到 10 只指定时间，假定日期为 2000-01-01。格式 11 的字符串 'now' 被转换为从当前正在使用的 sqlite3_vfs 对象的 xCurrentTime 方法获得的当前日期和时间。在同一个 sqlite3_step() 调用中，对日期和时间函数的 'now' 参数总是返回完全相同的值。使用 [协调世界时（UTC）](http://en.wikipedia.org/wiki/Coordinated_Universal_Time)。格式 12 是 [朱利安日期](http://en.wikipedia.org/wiki/Julian_day) 表示为整数或浮点值。如果紧随其后的是 'auto' 或 'unixepoch' 修饰符，格式 12 也可以解释为 Unix 时间戳。

格式 2 到 10 可能后跟形如 "*[+-]HH:MM*" 或只是 "*Z*" 的时区指示器。日期和时间函数内部使用 UTC 或 "zulu" 时间，因此 "Z" 后缀是一个空操作。任何非零的 "HH:MM" 后缀都会从指定的日期和时间中减去，以计算 zulu 时间。例如，以下所有时间值都是等效的：

> 2013-10-07 08:23:19.120  
> 
> 2013-10-07T08:23:19.120Z  
> 
> 2013-10-07 04:23:19.120-04:00  
> 
> 2456572.84952685  

在格式 4、7 和 10 中，小数秒值 SS.SSS 可以有一个或多个小数点后的数字。示例中只显示三位数字，因为只有前三位数字对结果有意义，但输入字符串可以有少于或多于三位数的小数位数，日期/时间函数仍然可以正确操作。类似地，格式 12 显示为 10 位有效数字，但日期/时间函数实际上接受表示朱利安日期号的任意多或少位数的数字。  

在除 timediff() 外的所有函数中，时间值（及所有修饰符）可以省略，此时假定时间值为 'now'。  

# 3\. 修饰符  

对于除 timediff() 外的所有日期/时间函数，时间值参数可以跟随零个或多个修改器，这些修改器改变日期和/或时间。每个修改器是应用于其左侧时间值的转换。修改器从左到右应用；顺序很重要。可用的修改器如下。  

1.  NNN 天  

1.  NNN 小时  

1.  NNN 分钟  

1.  NNN 秒  

1.  NNN 月  

1.  NNN 年  

1.  ±HH:MM  

1.  ±HH:MM:SS  

1.  ±HH:MM:SS.SSS  

1.  ±YYYY-MM-DD

1.  ±YYYY-MM-DD HH:MM  

1.  ±YYYY-MM-DD HH:MM:SS  

1.  ±YYYY-MM-DD HH:MM:SS.SSS  

1.  **ceiling**  

1.  **floor**  

1.  start of month  

1.  start of year  

1.  start of day  

1.  周几 N  

1.  unixepoch  

1.  julianday  

1.  auto  

1.  localtime  

1.  utc  

1.  subsec  

1.  subsecond  

前十三个修饰符（1 到 13）将指定的时间添加到左侧参数指定的日期和时间中。修饰符名字的结尾 's' 字符在修饰符 1 到 6 中是可选的。NNN 值可以是任意浮点数，带有可选的 '+' 或 '-' 前缀。  

**时间偏移修饰符**（7 到 13）通过指定的年、月、日、小时、分钟和/或秒数移动时间值。格式 10 到 13 需要初始的 '+' 或 '-' 符号，但格式 7、8 和 9 则是可选的。更改从左到右应用。首先按 YYYY 移动年份，然后按 MM 移动月份，接着按 DD 移动日期，依此类推。函数 timediff(A,B) 返回格式 13 中的时间偏移，将时间值 B 调整到 A。  

由于每个月或年的长度从一个月或年变化到下一个月或年，通过月份和/或年份移动日期时可能会出现歧义。例如，2024-02-29 之后一年的日期是什么？是 2025-02-28 还是 2025-03-01？或者是 2023-12-31 之后两个月的日期是什么？是 2024-02-29 还是 2024-03-02？如何解决这种歧义还没有共识，因此提供了 "**ceiling**" 和 "**floor**" 修饰符（14 和 15）供程序员选择。如果时间偏移后的下一个修饰符是 "ceiling"，则通过选择较晚的日期来解决任何日期的歧义。 "floor" 修饰符通过将日期解析为上个月的最后一天来解决歧义。默认行为是 "ceiling"。  

"**start of**" 修饰符（16 到 18）将日期向前移动到所在月份、年份或日的开头。

"**weekday**" 修饰符会将日期前进到下一个周几为 N 的日期，如果有必要的话。星期日为 0，星期一为 1，以此类推。如果日期已经是所需的周几，则 "weekday" 修饰符不会改变日期。

"**unixepoch**"（20）修饰符仅在紧跟格式为 DDDDDDDDDD 的时间值后面时起作用。此修饰符使 DDDDDDDDDD 被解释为 [Unix 时间](http://en.wikipedia.org/wiki/Unix_time) - 自 1970 年以来的秒数。如果 "unixepoch" 修饰符未紧跟形如表示自 1970 年以来秒数的 DDDDDDDDDD 的时间值，或者其他修饰符分隔 "unixepoch" 修饰符与前述 DDDDDDDDDD，则行为未定义。

"**julianday**" 修饰符必须紧跟格式为 DDDDDDDDD 的初始时间值后面。对 'julianday' 修饰符的任何其他使用都是错误的，并导致函数返回 NULL。'julianday' 修饰符强制将时间值数字解释为儒略日数。由于这是默认行为，'julianday' 修饰符几乎只是一个空操作。唯一的区别在于添加 'julianday' 强制使用 DDDDDDDDD 时间值格式，并在使用任何其他时间值格式时返回 NULL。

"**auto**" 修饰符必须紧跟在初始时间值后面。如果时间值是数字（格式为 DDDDDDDDDD），则 'auto' 修饰符会根据其大小将时间值解释为儒略日数或 Unix 时间戳。如果值在 0.0 到 5373484.499999 之间，则被解释为儒略日数（对应日期范围为 -4713-11-24 12:00:00 到 9999-12-31 23:59:59，包括边界）。对于超出有效儒略日数范围但在 -210866760000 到 253402300799 范围内的数值，'auto' 修饰符会将其解释为 Unix 时间戳。其他数值超出范围且会导致 NULL 返回。对于 ISO 8601 文本时间值，'auto' 修饰符不起作用。"auto" 修饰符设计用于处理即使在数据库文件中未知存储哪种时间值格式，或者同一列在不同行中存储不同格式的时间值的情况下。'auto' 修饰符会自动选择适当的格式。然而，存在一些歧义。1970 年的前 63 天的 Unix 时间戳将被解释为儒略日数。当数据集保证不包含该范围内的日期时，'auto' 修饰符非常有用，但在可能使用 1970 年开头几个月日期的应用中应避免使用。

"**localtime**"修饰符假定其左侧的时间值为协调世界时（UTC），并调整该时间值以使其为本地时间。如果"localtime"跟在非 UTC 时间之后，则行为是未定义的。"utc"修饰符是"localtime"的反义。"utc"假定其左侧的时间值为本地时区，并调整该时间值为 UTC。如果左侧的时间不在本地时间，则"utc"的结果是未定义的。

"**subsecond**"修饰符（也可以简写为"**subsec**"）可以增加 datetime()、time()和 unixepoch()的输出分辨率，以及 strftime()中"%s"格式字符串的精度。该修饰符对其他日期/时间函数没有影响。当前的实现将分辨率从秒增加到毫秒，但是未来 SQLite 版本可能会进一步增加分辨率。当使用"subsec"修饰符与 datetime()或 time()时，末尾的秒字段后面跟有小数点和一个或多个数字，以显示分数秒。当使用"subsec"修饰符与 unixepoch()时，结果是一个浮点值，表示自 1970-01-01 以来的秒数和分数秒。"subsecond"和"subsec"修饰符具有特殊属性，它们可以作为日期/时间函数的第一个参数（或作为 strftime()的格式字符串后的第一个参数）。当这种情况发生时，通常在第一个参数中的时间值被理解为"now"。例如，获取具有毫秒精度的当前时间戳的快捷方法是：

> SELECT unixepoch('subsec');

# 4\. 示例

计算当前日期。

> SELECT date();

计算当前月份的最后一天。

> SELECT date('now','start of month','+1 month','-1 day');

计算给定 Unix 时间戳 1092941466 的日期和时间。

> SELECT datetime(1092941466, 'unixepoch');
> 
> SELECT datetime(1092941466, 'auto'); -- 早期 1970 年不起作用！

计算给定 Unix 时间戳 1092941466 的日期和时间，并补偿您的本地时区。

> SELECT datetime(1092941466, 'unixepoch', 'localtime');

计算当前 Unix 时间戳。

> SELECT unixepoch();
> 
> SELECT strftime('%s');

计算自签署美国独立宣言以来的天数。

> SELECT julianday('now') - julianday('1776-07-04');

计算自 2004 年特定时刻以来的秒数：

> SELECT unixepoch() - unixepoch('2004-01-01 02:34:56');

计算当年 10 月第一个星期二的日期。

> SELECT date('now','start of year','+9 months','weekday 2');

计算自 Unix 纪元以来以毫秒为精度的秒数：

> SELECT (julianday('now') - 2440587.5)*86400.0;
> 
> SELECT unixepoch('now','subsec');

计算亚伯拉罕·林肯如果今天还活着会多大年纪：

> SELECT timediff('now','1809-02-12');

# 5\. 警告与错误

本地时间的计算严重依赖于政治家的心情，因此很难确保所有地区的正确性。在这个实现中，使用标准的 C 库函数 localtime_r()来辅助计算本地时间。localtime_r() C 函数通常只适用于 1970 年到 2037 年之间的年份。对于超出此范围的日期，SQLite 尝试将年份映射到此范围内的等效年份，进行计算，然后再映射回原来的年份。

这些函数仅适用于 0000-01-01 00:00:00 到 9999-12-31 23:59:59 之间的日期（儒略日号 1721059.5 到 5373484.5）。超出此范围的日期，这些函数的结果是未定义的。

非 Vista 版本的 Windows 平台仅支持一组夏令时规则。Vista 仅支持两组。因此，在这些平台上，历史夏令时计算将不正确。例如，在美国，2007 年夏令时规则发生了变化。非 Vista 版本的 Windows 平台将新的 2007 年夏令时规则应用到所有以前的年份。Vista 在回溯到 1986 年时做得比较好，当时规则也发生了变化。

所有内部计算都假设是[格里高利历](http://en.wikipedia.org/wiki/Gregorian_calendar)系统。它们还假设每天确切地是 86400 秒长，不考虑闰秒。
