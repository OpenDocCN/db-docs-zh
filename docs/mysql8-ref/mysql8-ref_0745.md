# 13.2.4 年份类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/year.html`](https://dev.mysql.com/doc/refman/8.0/en/year.html)

`YEAR`类型是用于表示年份值的 1 字节类型。它可以声明为带有 4 个字符的隐式显示宽度的`YEAR`，或者等效地声明为带有显式显示宽度的`YEAR(4)`。

注意

截至 MySQL 8.0.19，带有显式显示宽度的`YEAR(4)`数据类型已被弃用，您应该期望在未来的 MySQL 版本中删除对其的支持。取而代之的是，使用没有显示宽度的`YEAR`，其含义相同。

MySQL 8.0 不支持在旧版本 MySQL 中允许的 2 位`YEAR(2)`数据类型。有关转换为 4 位`YEAR`的说明，请参阅 2 位 YEAR(2)的限制和迁移到 4 位 YEAR，在 MySQL 5.7 参考手册中。

MySQL 以*`YYYY`*格式显示`YEAR`值，范围为`1901`到`2155`，以及`0000`。

`YEAR`接受各种格式的输入值：

+   作为范围在`'1901'`到`'2155'`之间的 4 位数字字符串。

+   作为范围在`1901`到`2155`之间的 4 位数字。

+   作为范围在`'0'`到`'99'`之间的 1 位或 2 位字符串。MySQL 将范围在`'0'`到`'69'`和`'70'`到`'99'`之间的值转换为范围在`2000`到`2069`和`1970`到`1999`之间的`YEAR`值。

+   作为范围在`0`到`99`之间的 1 位或 2 位数字。MySQL 将范围在`1`到`69`和`70`到`99`之间的值转换为范围在`2001`到`2069`和`1970`到`1999`之间的`YEAR`值。

    插入数字`0`的结果显示值为`0000`，内部值为`0000`。要插入零并将其解释为`2000`，请将其指定为字符串`'0'`或`'00'`。

+   作为返回在`YEAR`上下文中可接受值的函数的结果，例如`NOW()`。

如果未启用严格的 SQL 模式，MySQL 会将无效的`YEAR`值转换为`0000`。在严格的 SQL 模式下，尝试插入无效的`YEAR`值会产生错误。

另请参阅第 13.2.9 节，“日期中的 2 位年份”。