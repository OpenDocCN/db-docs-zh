# 13.1.2 整数类型（精确值）- INTEGER、INT、SMALLINT、TINYINT、MEDIUMINT、BIGINT

> 原文：[`dev.mysql.com/doc/refman/8.0/en/integer-types.html`](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)

MySQL 支持 SQL 标准整数类型`INTEGER`（或`INT`）和`SMALLINT`。作为对标准的扩展，MySQL 还支持整数类型`TINYINT`、`MEDIUMINT`和`BIGINT`。以下表格显示了每种整数类型所需的存储空间和范围。

**表 13.1 MySQL 支持的整数类型所需的存储空间和范围**

| 类型 | 存储空间（字节） | 最小有符号值 | 最小无符号值 | 最大有符号值 | 最大无符号值 |
| --- | --- | --- | --- | --- | --- |
| `TINYINT` | 1 | `-128` | `0` | `127` | `255` |
| `SMALLINT` | 2 | `-32768` | `0` | `32767` | `65535` |
| `MEDIUMINT` | 3 | `-8388608` | `0` | `8388607` | `16777215` |
| `INT` | 4 | `-2147483648` | `0` | `2147483647` | `4294967295` |
| `BIGINT` | 8 | `-2⁶³` | `0` | `2⁶³-1` | `2⁶⁴-1` |
