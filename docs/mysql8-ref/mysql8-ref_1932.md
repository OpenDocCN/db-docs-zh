# 28.3.37 信息模式 ST_UNITS_OF_MEASURE 表

> [`dev.mysql.com/doc/refman/8.0/en/information-schema-st-units-of-measure-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-st-units-of-measure-table.html)

`ST_UNITS_OF_MEASURE` 表（自 MySQL 8.0.14 起可用）提供了关于 `ST_Distance()` 函数可接受单位的信息。

`ST_UNITS_OF_MEASURE` 表包含以下列：

+   `UNIT_NAME`

    单位的名称。

+   `UNIT_TYPE`

    单位类型（例如，`线性`）。

+   `CONVERSION_FACTOR`

    用于内部计算的转换因子。

+   `DESCRIPTION`

    单位的描述。
