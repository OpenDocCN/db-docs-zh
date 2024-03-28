# 13.1.1 数字数据类型语法

> 原文：[`dev.mysql.com/doc/refman/8.0/en/numeric-type-syntax.html`](https://dev.mysql.com/doc/refman/8.0/en/numeric-type-syntax.html)

对于整数数据类型，*`M`* 表示最小显示宽度。最大显示宽度为 255。显示宽度与类型可以存储的值范围无关，如 第 13.1.6 节，“数字类型属性” 中所述。

对于浮点和定点数据类型，*`M`* 是可以存储的总位数。

截至 MySQL 8.0.17 版本，整数数据类型的显示宽度属性已被弃用；您应该预期在未来的 MySQL 版本中将其移除。

如果为数字列指定了 `ZEROFILL`，MySQL 会自动向列添加 `UNSIGNED` 属性。

截至 MySQL 8.0.17 版本，`ZEROFILL` 属性对于数字数据类型已被弃用；您应该预期在未来的 MySQL 版本中将其移除。考虑使用其他方法来产生此属性的效果。例如，应用程序可以使用 `LPAD()` 函数将数字零填充到所需宽度，或者它们可以将格式化的数字存储在 `CHAR` 列中。

允许 `UNSIGNED` 属性的数字数据类型也允许 `SIGNED`。但是，这些数据类型默认为有符号，因此 `SIGNED` 属性没有效果。

截至 MySQL 8.0.17 版本，`UNSIGNED` 属性对于 `FLOAT`、`DOUBLE` 和 `DECIMAL`（以及任何同义词）列已被弃用；您应该预期在未来的 MySQL 版本中将其移除。考虑为这些列使用简单的 `CHECK` 约束代替。

`SERIAL` 是 `BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE` 的别名。

在整数列的定义中，`SERIAL DEFAULT VALUE` 是 `NOT NULL AUTO_INCREMENT UNIQUE` 的别名。

警告

当您在一个类型为 `UNSIGNED` 的整数值之间进行减法运算时，结果是无符号的，除非启用了 `NO_UNSIGNED_SUBTRACTION` SQL 模式。参见 第 14.10 节，“转换函数和运算符”。

+   [`BIT[(*`M`*)]`](bit-type.html "13.1.5 位值类型 - BIT")

    位值类型。*`M`* 表示每个值的位数，从 1 到 64。如果省略 *`M`*，默认值为 1。

+   [`TINYINT[(*`M`*)] [UNSIGNED] [ZEROFILL]`](integer-types.html "13.1.2 整数类型（精确值） - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")

    非常小的整数。有符号范围为 `-128` 到 `127`。无符号范围为 `0` 到 `255`。

+   `BOOL`, `BOOLEAN`

    这些类型是`TINYINT(1)`的同义词。零值被视为假。非零值被视为真：

    ```sql
    mysql> SELECT IF(0, 'true', 'false');
    +------------------------+
    | IF(0, 'true', 'false') |
    +------------------------+
    | false                  |
    +------------------------+

    mysql> SELECT IF(1, 'true', 'false');
    +------------------------+
    | IF(1, 'true', 'false') |
    +------------------------+
    | true                   |
    +------------------------+

    mysql> SELECT IF(2, 'true', 'false');
    +------------------------+
    | IF(2, 'true', 'false') |
    +------------------------+
    | true                   |
    +------------------------+
    ```

    然而，值`TRUE`和`FALSE`仅仅是`1`和`0`的别名，如下所示：

    ```sql
    mysql> SELECT IF(0 = FALSE, 'true', 'false');
    +--------------------------------+
    | IF(0 = FALSE, 'true', 'false') |
    +--------------------------------+
    | true                           |
    +--------------------------------+

    mysql> SELECT IF(1 = TRUE, 'true', 'false');
    +-------------------------------+
    | IF(1 = TRUE, 'true', 'false') |
    +-------------------------------+
    | true                          |
    +-------------------------------+

    mysql> SELECT IF(2 = TRUE, 'true', 'false');
    +-------------------------------+
    | IF(2 = TRUE, 'true', 'false') |
    +-------------------------------+
    | false                         |
    +-------------------------------+

    mysql> SELECT IF(2 = FALSE, 'true', 'false');
    +--------------------------------+
    | IF(2 = FALSE, 'true', 'false') |
    +--------------------------------+
    | false                          |
    +--------------------------------+
    ```

    最后两个语句显示的结果是因为`2`既不等于`1`也不等于`0`。

+   [`SMALLINT[(*`M`*)] [UNSIGNED] [ZEROFILL]`](integer-types.html "13.1.2 整数类型（精确值） - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")

    一个小整数。有符号范围是`-32768`到`32767`。无符号范围是`0`到`65535`。

+   [`MEDIUMINT[(*`M`*)] [UNSIGNED] [ZEROFILL]`](integer-types.html "13.1.2 整数类型（精确值） - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")

    一个中等大小的整数。有符号范围是`-8388608`到`8388607`。无符号范围是`0`到`16777215`。

+   [`INT[(*`M`*)] [UNSIGNED] [ZEROFILL]`](integer-types.html "13.1.2 整数类型（精确值） - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")

    一个正常大小的整数。有符号范围是`-2147483648`到`2147483647`。无符号范围是`0`到`4294967295`。

+   [`INTEGER[(*`M`*)] [UNSIGNED] [ZEROFILL]`](integer-types.html "13.1.2 整数类型（精确值） - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")

    这种类型是`INT`的同义词。

+   [`BIGINT[(*`M`*)] [UNSIGNED] [ZEROFILL]`](integer-types.html "13.1.2 整数类型（精确值） - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")

    一个大整数。有符号范围是`-9223372036854775808`到`9223372036854775807`。无符号范围是`0`到`18446744073709551615`。

    `SERIAL`是`BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE`的别名。

    关于`BIGINT`列，有一些需要注意的事项：

    +   所有算术运算都是使用有符号的`BIGINT`或`DOUBLE`值进行的，因此除了使用位函数外，不应使用大于`9223372036854775807`（63 位）的无符号大整数！如果这样做，由于将`BIGINT`值转换为`DOUBLE`时的四舍五入误差，结果中的一些最后几位可能是错误的。

        MySQL 可以在以下情况下处理`BIGINT`：

        +   当使用整数来存储大无符号值在`BIGINT`列中。

        +   在`MIN(*`col_name`*)`或`MAX(*`col_name`*)`中，其中*`col_name`*指的是`BIGINT`列。

        +   当使用操作符（`+`、`-`、`*`等）其中两个操作数都是整数时。

    +   您始终可以通过使用字符串将精确整数值存储在`BIGINT`列中来存储它。在这种情况下，MySQL 执行一个涉及没有中间双精度表示的字符串到数字的转换。

    +   当两个操作数都是整数值时，`-`、`+`和`*`运算符使用`BIGINT`算术。这意味着如果您将两个大整数相乘（或从返回整数的函数得到的结果），当结果大于`9223372036854775807`时，您可能会得到意外的结果。

+   [`DECIMAL[(*`M`*[,*`D`*])] [UNSIGNED] [ZEROFILL]`](fixed-point-types.html "13.1.3 定点类型（精确值） - DECIMAL, NUMERIC")

    一种紧凑的“精确”固定点数。*`M`*是总位数（精度），*`D`*是小数点后的位数（比例）。小数点和（对于负数）`-`符号不计入*`M`*。如果*`D`*为 0，则值没有小数点或小数部分。`DECIMAL`的最大数字（*`M`*）为 65。支持的小数位数（*`D`*）的最大值为 30。如果省略*`D`*，默认值为 0。如果省略*`M`*，默认值为 10。（`DECIMAL`文本的长度也有限制；请参阅第 14.24.3 节，“表达式处理”。）

    如果指定了`UNSIGNED`，则不允许负值。截至 MySQL 8.0.17，对于`DECIMAL`（以及任何同义词）列，`UNSIGNED`属性已被弃用；您应该期望在将来的 MySQL 版本中删除对其的支持。考虑为这些列使用简单的`CHECK`约束。

    所有对`DECIMAL`列的基本计算（`+`，`-`，`*`，`/`）都以 65 位数字的精度进行。

+   [`DEC[(*`M`*[,*`D`*])] [UNSIGNED] [ZEROFILL]`](fixed-point-types.html "13.1.3 固定点类型（精确值） - DECIMAL, NUMERIC")，[`NUMERIC[(*`M`*[,*`D`*])] [UNSIGNED] [ZEROFILL]`](fixed-point-types.html "13.1.3 固定点类型（精确值） - DECIMAL, NUMERIC")，[`FIXED[(*`M`*[,*`D`*])] [UNSIGNED] [ZEROFILL]`](fixed-point-types.html "13.1.3 固定点类型（精确值） - DECIMAL, NUMERIC")

    这些类型是`DECIMAL`的同义词。`FIXED`同义词可用于与其他数据库系统的兼容性。

+   [`FLOAT[(*`M`*,*`D`*)] [UNSIGNED] [ZEROFILL]`](floating-point-types.html "13.1.4 浮点类型（近似值） - FLOAT, DOUBLE")

    一个小（单精度）浮点数。允许的值为`-3.402823466E+38`到`-1.175494351E-38`，`0`，和`1.175494351E-38`到`3.402823466E+38`。这些是基于 IEEE 标准的理论极限。实际范围可能会略小，取决于您的硬件或操作系统。

    *`M`*是总位数，*`D`*是小数点后的位数。如果省略*`M`*和*`D`*，则值将存储到硬件允许的极限。单精度浮点数精确到大约 7 位小数。

    `FLOAT(*M*,*D*)`是一个非标准的 MySQL 扩展。截至 MySQL 8.0.17 版本，此语法已被弃用，您应该期待在未来的 MySQL 版本中移除对其的支持。

    `UNSIGNED`，如果指定，将禁止负值。截至 MySQL 8.0.17 版本，对于`FLOAT`类型的列，`UNSIGNED`属性已被弃用（以及任何同义词），您应该期待在未来的 MySQL 版本中移除对其的支持。考虑为这些列使用简单的`CHECK`约束。

    在 MySQL 中，所有计算都是以双精度进行的，因此使用`FLOAT`可能会导致一些意外的问题。参见第 B.3.4.7 节，“解决没有匹配行的问题”。

+   [`FLOAT(*p*) [UNSIGNED] [ZEROFILL]`](floating-point-types.html "13.1.4 浮点类型（近似值） - FLOAT, DOUBLE")

    一个浮点数。*p*表示位数精度，但 MySQL 仅使用此值来确定是否使用`FLOAT`或`DOUBLE`作为结果数据类型。如果*p*从 0 到 24，数据类型变为没有*M*或*D*值的`FLOAT`。如果*p*从 25 到 53，数据类型变为没有*M*或*D*值的`DOUBLE`。结果列的范围与本节前面描述的单精度`FLOAT`或双精度`DOUBLE`数据类型的范围相同。

    `UNSIGNED`，如果指定，将禁止负值。截至 MySQL 8.0.17 版本，对于`FLOAT`类型的列，`UNSIGNED`属性已被弃用（以及任何同义词），您应该期待在未来的 MySQL 版本中移除对其的支持。考虑为这些列使用简单的`CHECK`约束。

    `FLOAT(*p*)`语法是为了 ODBC 兼容性而提供的。

+   [`DOUBLE[(*M*,*D*)] [UNSIGNED] [ZEROFILL]`](floating-point-types.html "13.1.4 浮点类型（近似值） - FLOAT, DOUBLE")

    一个正常大小（双精度）的浮点数。允许的值为`-1.7976931348623157E+308`到`-2.2250738585072014E-308`，`0`，以及`2.2250738585072014E-308`到`1.7976931348623157E+308`。这些是基于 IEEE 标准的理论极限。实际范围可能会略小，取决于您的硬件或操作系统。

    *`M`*是总位数，*`D`*是小数点后的位数。如果省略了*`M`*和*`D`*，则值将存储到硬件允许的极限。双精度浮点数精确到大约 15 位小数。

    `DOUBLE(*`M`*,*`D`*)`是 MySQL 的非标准扩展。截至 MySQL 8.0.17 版本，此语法已被弃用，您应该期待在未来的 MySQL 版本中移除对其的支持。

    `UNSIGNED`，如果指定，将不允许负值。截至 MySQL 8.0.17 版本，对于`DOUBLE`类型的列，`UNSIGNED`属性已被弃用（以及任何同义词），您应该期待在未来的 MySQL 版本中移除对其的支持。考虑为这些列使用简单的`CHECK`约束。

+   [`DOUBLE PRECISION[(*`M`*,*`D`*)] [UNSIGNED] [ZEROFILL]`](floating-point-types.html "13.1.4 浮点类型（近似值） - FLOAT，DOUBLE")，[`REAL[(*`M`*,*`D`*)] [UNSIGNED] [ZEROFILL]`](floating-point-types.html "13.1.4 浮点类型（近似值） - FLOAT，DOUBLE")

    这些类型是`DOUBLE`的同义词。例外：如果启用了`REAL_AS_FLOAT` SQL 模式，则`REAL`是`FLOAT`的同义词，而不是`DOUBLE`。
