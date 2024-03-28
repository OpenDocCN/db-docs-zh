> 原文：[`dev.mysql.com/doc/refman/8.0/en/constant-folding-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/constant-folding-optimization.html)

#### 10.2.1.14 常量折叠优化

常量与列值之间的比较，其中常量值超出范围或与列类型不匹配，现在在查询优化期间处理一次，而不是在执行期间逐行处理。可以以这种方式处理的比较包括`>`, `>=`, `<`, `<=`, `<>`/`!=`, `=`, 和 `<=>`。

考虑以下语句创建的表：

```sql
CREATE TABLE t (c TINYINT UNSIGNED NOT NULL);
```

查询`SELECT * FROM t WHERE c < 256`中的`WHERE`条件包含整数常量 256，这对于`TINYINT UNSIGNED`列来说超出范围。以前，这是通过将两个操作数都视为较大类型来处理的，但现在，由于`c`的任何允许值都小于常量，因此`WHERE`表达式可以折叠为`WHERE 1`，使查询重写为`SELECT * FROM t WHERE 1`。

这使得优化器可以完全删除`WHERE`表达式。如果列`c`可为空（即仅定义为`TINYINT UNSIGNED`），则查询将被重写如下：

```sql
SELECT * FROM t WHERE ti IS NOT NULL
```

对比较支持的 MySQL 列类型的常量进行折叠如下：

+   **整数列类型。** 整数类型与以下类型的常量进行比较，如下所述：

    +   **整数值。** 如果常量超出列类型的范围，比较将折叠为`1`或`IS NOT NULL`，如已经显示的那样。

        如果常量是一个范围边界，比较将折叠为`=`。例如（使用已经定义的相同表）：

        ```sql
        mysql> EXPLAIN SELECT * FROM t WHERE c >= 255;
        *************************** 1\. row ***************************
                   id: 1
          select_type: SIMPLE
                table: t
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 5
             filtered: 20.00
                Extra: Using where 1 row in set, 1 warning (0.00 sec)

        mysql> SHOW WARNINGS;
        *************************** 1\. row ***************************
          Level: Note
           Code: 1003
        Message: /* select#1 */ select `test`.`t`.`ti` AS `ti` from `test`.`t` where (`test`.`t`.`ti` = 255) 1 row in set (0.00 sec)
        ```

    +   **浮点或定点值。** 如果常量是十进制类型之一（如`DECIMAL`，`REAL`，`DOUBLE`或`FLOAT`）并且具有非零小数部分，则不能相等；相应地折叠。对于其他比较，根据符号四舍五入到整数值，然后执行范围检查并按照已经描述的方式处理整数-整数比较。

        无法表示为`DECIMAL`的`REAL`值将四舍五入为.01 或-.01，然后作为`DECIMAL`处理。

    +   **字符串类型。** 尝试将字符串值解释为整数类型，然后将比较处理为整数值之间的比较。如果失败，则尝试将值处理为`REAL`。

+   **DECIMAL 或 REAL 列。** 十进制类型与以下类型的常量进行比较，如下所述：

    +   **整数值。** 对列值的整数部分执行范围检查。如果没有折叠结果，将常量转换为与列值具有相同小数位数的`DECIMAL`，然后将其作为`DECIMAL`进行检查（见下文）。

    +   **DECIMAL 或 REAL 值。** 检查溢出（即常量的整数部分是否比列的十进制类型允许的更多位数）。如果是，则折叠。

        如果常量的有效小数位数多于列的类型，截断常量。如果比较运算符是`=`或`<>`，则折叠。如果运算符是`>=`或`<=`，由于截断而调整运算符。例如，如果列的类型是`DECIMAL(3,1)`，`SELECT * FROM t WHERE f >= 10.13`变为`SELECT * FROM t WHERE f > 10.1`。

        如果常量的小数位数少于列的类型，将其转换为具有相同位数的常量。对于`REAL`值的下溢（即，小数位数太少无法表示），将常量转换为十进制 0。

    +   **字符串值。** 如果值可以解释为整数类型，则将其处理为整数类型。否则，尝试将其处理为`REAL`。

+   **FLOAT 或 DOUBLE 列。** `FLOAT(*m*,*n*)`或`DOUBLE(*m*,*n*)`与常量的比较处理如下：

    如果值超出列的范围，折叠。

    如果值有超过*n*个小数位，截断，折叠时进行补偿。对于`=`和`<>`比较，按照之前描述的折叠为`TRUE`、`FALSE`或`IS [NOT] NULL`；对于其他运算符，调整运算符。

    如果值有超过`m`个整数位，折叠。

**限制。** 该优化不能用于以下情况：

1.  使用`BETWEEN`或`IN`进行比较。

1.  与`BIT`列或使用日期或时间类型的列。

1.  在准备语句的准备阶段，尽管可以在实际执行准备语句时进行优化阶段应用。这是因为在语句准备期间，常量的值尚未知晓。
