# 13.8 选择适合列的正确类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/choosing-types.html`](https://dev.mysql.com/doc/refman/8.0/en/choosing-types.html)

为了最佳存储，您应该尽量在所有情况下使用最精确的类型。例如，如果整数列用于范围在`1`到`99999`之间的值，`MEDIUMINT UNSIGNED`是最佳类型。在表示所有所需值的类型中，此类型使用的存储空间最少。

所有基本计算（`+`, `-`, `*`, 和 `/`）与`DECIMAL`列都以 65 位十进制（基数 10）数字的精度进行。参见 Section 13.1.1, “Numeric Data Type Syntax”。

如果精度不是太重要，或者速度是最高优先级，`DOUBLE` 类型可能已经足够好了。对于高精度，您可以随时转换为存储在`BIGINT`中的固定点类型。这使您可以使用 64 位整数进行所有计算，然后根据需要将结果转换回浮点值。
