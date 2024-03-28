# 28.3.20 INFORMATION_SCHEMA PARAMETERS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-parameters-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-parameters-table.html)

`PARAMETERS`表提供有关存储例程（存储过程和存储函数）的参数以及存储函数的返回值的信息。`PARAMETERS`表不包括内置（本机）函数或可加载函数。

`PARAMETERS`表具有以下列：

+   `SPECIFIC_CATALOG`

    包含参数的例程所属的目录的名称。该值始终为`def`。

+   `SPECIFIC_SCHEMA`

    包含参数的例程所属的模式（数据库）的名称。

+   `SPECIFIC_NAME`

    包含参数的例程名称。

+   `ORDINAL_POSITION`

    对于存储过程或函数的连续参数，`ORDINAL_POSITION`值为 1、2、3 等。对于存储函数，还有一行适用于函数返回值（由`RETURNS`子句描述）。返回值不是真正的参数，因此描述它的行具有以下独特特征：

    +   `ORDINAL_POSITION`值为 0。

    +   `PARAMETER_NAME`和`PARAMETER_MODE`值为`NULL`，因为返回值没有名称，模式也不适用。

+   `PARAMETER_MODE`

    参数的模式。该值为`IN`、`OUT`或`INOUT`之一。对于存储函数的返回值，该值为`NULL`。

+   `PARAMETER_NAME`

    参数的名称。对于存储函数的返回值，该值为`NULL`。

+   `DATA_TYPE`

    参数数据类型。

    `DATA_TYPE`值仅为类型名称，没有其他信息。`DTD_IDENTIFIER`值包含类型名称，可能还包含其他信息，如精度或长度。

+   `CHARACTER_MAXIMUM_LENGTH`

    对于字符串参数，以字符为单位的最大长度。

+   `CHARACTER_OCTET_LENGTH`

    对于字符串参数，以字节为单位的最大长度。

+   `NUMERIC_PRECISION`

    对于数值参数，数值精度。

+   `NUMERIC_SCALE`

    对于数值参数，数值刻度。

+   `DATETIME_PRECISION`

    对于时间参数，分数秒精度。

+   `CHARACTER_SET_NAME`

    对于字符串参数，字符集名称。

+   `COLLATION_NAME`

    对于字符串参数，排序名称。

+   `DTD_IDENTIFIER`

    参数数据类型。

    `DATA_TYPE`值仅为类型名称，没有其他信息。`DTD_IDENTIFIER`值包含类型名称，可能还包含其他信息，如精度或长度。

+   `ROUTINE_TYPE`

    对于存储过程，`PROCEDURE`，对于存储函数，`FUNCTION`。
