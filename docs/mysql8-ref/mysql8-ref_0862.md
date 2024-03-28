# 14.19 聚合函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/aggregate-functions-and-modifiers.html`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions-and-modifiers.html)

14.19.1 聚合函数描述

14.19.2 GROUP BY 修饰符

14.19.3 MySQL 对 GROUP BY 的处理

14.19.4 功能依赖的检测

聚合函数操作在一组值上。它们通常与`GROUP BY`子句一起使用，将值分组为子集。本节描述了大多数聚合函数。有关操作几何值的聚合函数的信息，请参见第 14.16.12 节，“空间聚合函数”。
