# 11.1.2 数值文字

> 原文：[`dev.mysql.com/doc/refman/8.0/en/number-literals.html`](https://dev.mysql.com/doc/refman/8.0/en/number-literals.html)

数字文字包括精确值（整数和 `DECIMAL` - DECIMAL, NUMERIC")）文字和近似值（浮点）文字。

整数表示为一系列数字。数字可以包括 `.` 作为小数分隔符。数字可以以 `-` 或 `+` 开头，表示负值或正值。用尾数和指数表示的数字是近似值数字。

精确值数值文字具有整数部分或小数部分，或两者兼有。它们可以带符号。例如：`1`，`.2`，`3.4`，`-5`，`-6.78`，`+9.10`。

近似值数值文字用科学计数法表示，具有尾数和指数。尾数和/或指数部分可以带符号。例如：`1.2E3`，`1.2E-3`，`-1.2E3`，`-1.2E-3`。

看起来相似的两个数字可能被视为不同。例如，`2.34` 是一个精确值（固定点）数字，而 `2.34E0` 是一个近似值（浮点）数字。

`DECIMAL` - DECIMAL, NUMERIC") 数据类型是一个固定点类型，计算是精确的。在 MySQL 中，`DECIMAL` - DECIMAL, NUMERIC") 类型有几个同义词：`NUMERIC` - DECIMAL, NUMERIC")，`DEC` - DECIMAL, NUMERIC")，`FIXED` - DECIMAL, NUMERIC")。整数类型也是精确值类型。有关精确值计算的更多信息，请参见 第 14.24 节，“精度数学”。

`FLOAT` - FLOAT, DOUBLE") 和 `DOUBLE` - FLOAT, DOUBLE") 数据类型是浮点类型，计算是近似的。在 MySQL 中，与 `FLOAT` - FLOAT, DOUBLE") 或 `DOUBLE` - FLOAT, DOUBLE") 同义的类型有 `DOUBLE PRECISION` - FLOAT, DOUBLE") 和 `REAL` - FLOAT, DOUBLE")。

整数可以在浮点上下文中使用；它被解释为等效的浮点数。
