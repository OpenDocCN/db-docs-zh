# 14.24.1 数值类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/precision-math-numbers.html`](https://dev.mysql.com/doc/refman/8.0/en/precision-math-numbers.html)

精确值操作的精度数学范围包括精确值数据类型（整数和`DECIMAL` - DECIMAL, NUMERIC") 类型）和精确值数值文字。近似值数据类型和数值文字被处理为浮点数。

精确值数值文字具有整数部分、小数部分或两者兼有。它们可以带有符号。例如：`1`、`.2`、`3.4`、`-5`、`-6.78`、`+9.10`。

近似值数值文字用科学计数法表示，包括尾数和指数。尾数和/或指数部分可以带有符号。例如：`1.2E3`、`1.2E-3`、`-1.2E3`、`-1.2E-3`。

看似相似的两个数字可能会被处理得不同。例如，`2.34`是一个精确值（定点）数字，而`2.34E0`是一个近似值（浮点）数字。

`DECIMAL` - DECIMAL, NUMERIC") 数据类型是一个定点类型，计算是精确的。在 MySQL 中，`DECIMAL` - DECIMAL, NUMERIC") 类型有几个同义词：`NUMERIC` - DECIMAL, NUMERIC")、`DEC` - DECIMAL, NUMERIC")、`FIXED` - DECIMAL, NUMERIC")。整数类型也是精确值类型。

`FLOAT` - FLOAT, DOUBLE") 和 `DOUBLE` - FLOAT, DOUBLE") 数据类型是浮点类型，计算是近似的。在 MySQL 中，与 `FLOAT` - FLOAT, DOUBLE") 或 `DOUBLE` - FLOAT, DOUBLE") 同义的类型有 `DOUBLE PRECISION` - FLOAT, DOUBLE") 和 `REAL` - FLOAT, DOUBLE")。
