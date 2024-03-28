# 13.1 数值数据类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/numeric-types.html`](https://dev.mysql.com/doc/refman/8.0/en/numeric-types.html)

13.1.1 数值数据类型语法

13.1.2 整数类型（精确值） - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT

13.1.3 定点类型（精确值） - DECIMAL, NUMERIC

13.1.4 浮点类型（近似值） - FLOAT, DOUBLE

13.1.5 位值类型 - BIT

13.1.6 数值类型属性

13.1.7 超出范围和溢出处理

MySQL 支持所有标准 SQL 数值数据类型。这些类型包括精确数值数据类型（`INTEGER`, `SMALLINT`, `DECIMAL`和`NUMERIC`)，以及近似数值数据类型（`FLOAT`, `REAL`和`DOUBLE PRECISION`）。关键字`INT`是`INTEGER`的同义词，关键字`DEC`和`FIXED`是`DECIMAL`的同义词。MySQL 将`DOUBLE`视为`DOUBLE PRECISION`的同义词（非标准扩展）。MySQL 还将`REAL`视为`DOUBLE PRECISION`的同义词（非标准变体），除非启用了`REAL_AS_FLOAT` SQL 模式。

`BIT`数据类型存储位值，并支持`MyISAM`、`MEMORY`、`InnoDB`和`NDB`表。

有关 MySQL 如何处理超出范围值分配给列和在表达式评估期间的溢出的信息，请参阅第 13.1.7 节，“超出范围和溢出处理”。

有关数值数据类型的存储要求的信息，请参阅第 13.7 节，“数据类型存储要求”。

对于操作数值的函数描述，请参阅第 14.6 节，“数值函数和运算符”。对于数值操作数计算结果使用的数据类型取决于操作数的类型和执行的操作。更多信息，请参阅第 14.6.1 节，“算术运算符”。
