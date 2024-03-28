> 原文：[`dev.mysql.com/doc/refman/8.0/en/window-function-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/window-function-optimization.html)

#### 10.2.1.21 窗口函数优化

窗口函数会影响优化器考虑的策略：

+   如果子查询具有窗口函数，则禁用对子查询的派生表合并。子查询总是被实体化。

+   半连接不适用于窗口函数优化，因为半连接适用于`WHERE`和`JOIN ... ON`中的子查询，这些子查询不能包含窗口函数。

+   优化器按顺序处理具有相同排序要求的多个窗口，因此对于第一个窗口之后的窗口，可以跳过排序。

+   优化器不会尝试合并可以在单个步骤中评估的窗口（例如，当多个`OVER`子句包含相同的窗口定义时）。解决方法是在`WINDOW`子句中定义窗口，并在`OVER`子句中引用窗口名称。

未作为窗口函数使用的聚合函数在可能的最外层查询中进行聚合。例如，在这个查询中，MySQL 看到`COUNT(t1.b)`是一个不能存在于外部查询中的东西，因为它在`WHERE`子句中的位置：

```sql
SELECT * FROM t1 WHERE t1.a = (SELECT COUNT(t1.b) FROM t2);
```

因此，MySQL 在子查询中进行聚合，将`t1.b`视为常量，并返回`t2`行的计数。

将`WHERE`替换为`HAVING`会导致错误：

```sql
mysql> SELECT * FROM t1 HAVING t1.a = (SELECT COUNT(t1.b) FROM t2);
ERROR 1140 (42000): In aggregated query without GROUP BY, expression #1
of SELECT list contains nonaggregated column 'test.t1.a'; this is
incompatible with sql_mode=only_full_group_by
```

错误发生是因为`COUNT(t1.b)`可能存在于`HAVING`中，因此使外部查询聚合。

窗口函数（包括作为窗口函数使用的聚合函数）没有前面的复杂性。它们总是在写入它们的子查询中进行聚合，而不是在外部查询中进行。

窗口函数的评估可能会受到`windowing_use_high_precision`系统变量值的影响，该变量确定是否计算窗口操作时不损失精度。默认情况下，`windowing_use_high_precision`是启用的。

对于一些移动框架聚合，可以应用逆聚合函数来从聚合中移除值。这可以提高性能，但可能会损失精度。例如，将一个非常小的浮点值加到一个非常大的值上会导致这个非常小的值被这个大值“隐藏”。当稍后对大值进行反转时，小值的效果就会丢失。

由于逆聚合而导致精度丢失仅适用于浮点（近似值）数据类型的操作。对于其他类型，逆聚合是安全的；这包括允许有小数部分但是精确值类型的`DECIMAL`。

为了更快地执行，MySQL 总是在安全的情况下使用逆聚合：

+   对于浮点值，逆聚合并不总是安全的，可能会导致精度丢失。默认情况下是避免逆聚合，这样会更慢但保留精度。如果可以为了速度而牺牲安全性，可以禁用`windowing_use_high_precision`以允许逆聚合。

+   对于非浮点数据类型，逆聚合始终是安全的，并且无论`windowing_use_high_precision`的值如何都会使用。

+   `windowing_use_high_precision`对于`MIN()`和`MAX()`没有影响，在任何情况下都不使用逆聚合。

对于方差函数`STDDEV_POP()`, `STDDEV_SAMP()`, `VAR_POP()`, `VAR_SAMP()`及其同义词的评估，评估可以在优化模式或默认模式下进行。优化模式可能在最后几位有效数字上产生略有不同的结果。如果这种差异是可以接受的，可以禁用`windowing_use_high_precision`以允许优化模式。

对于`EXPLAIN`，窗口执行计划信息在传统输出格式中显示的内容太多。要查看窗口信息，请使用`EXPLAIN FORMAT=JSON`并查找`windowing`元素。
