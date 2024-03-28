# 13.1.4 浮点类型（近似值） - FLOAT, DOUBLE

> 原文：[`dev.mysql.com/doc/refman/8.0/en/floating-point-types.html`](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html)

`FLOAT` 和 `DOUBLE` 类型表示近似数值数据。MySQL 使用四个字节来存储单精度值，使用八个字节来存储双精度值。

对于 `FLOAT`，SQL 标准允许在关键字 `FLOAT` 后面的括号中指定精度（但不是指数范围）的可选规范，即 `FLOAT(*`p`*)`。MySQL 也支持这种可选精度规范，但在 `FLOAT(*`p`*)` 中的精度值仅用于确定存储大小。精度从 0 到 23 会导致一个 4 字节的单精度 `FLOAT` 列。精度从 24 到 53 会导致一个 8 字节的双精度 `DOUBLE` 列。

MySQL 允许非标准的语法：`FLOAT(*`M`*,*`D`*)` 或 `REAL(*`M`*,*`D`*)` 或 `DOUBLE PRECISION(*`M`*,*`D`*)`。这里，`(*`M`*,*`D`*)` 意味着值可以以最多 *`M`* 位数字的形式存储，其中 *`D`* 位可以在小数点后面。例如，定义为 `FLOAT(7,4)` 的列显示为 `-999.9999`。当存储值时，MySQL 进行四舍五入，因此如果你将 `999.00009` 插入到 `FLOAT(7,4)` 列中，近似结果是 `999.0001`。

从 MySQL 8.0.17 开始，非标准的 `FLOAT(*`M`*,*`D`*)` 和 `DOUBLE(*`M`*,*`D`*)` 语法已被弃用，你应该预期在未来的 MySQL 版本中将不再支持它。

因为浮点值是近似值，而不是精确值，试图在比较中将它们视为精确值可能会导致问题。它们也受平台或实现的依赖性影响。更多信息，请参见 Section B.3.4.8, “浮点值的问题”。

为了最大的可移植性，需要存储近似数值数据的代码应该使用 `FLOAT` 或 `DOUBLE PRECISION`，不需要指定精度或数字位数。
