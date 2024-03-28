> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-format-bytes.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-format-bytes.html)

#### 30.4.5.3 `format_bytes()` 函数

注意

截至 MySQL 8.0.16 版本，`format_bytes()`` 函数") 已被弃用，并可能在未来的 MySQL 版本中移除。使用该函数的应用程序应迁移到使用内置的 `FORMAT_BYTES()` 函数。参见 Section 14.21, “Performance Schema Functions”

给定一个字节计数，将其转换为人类可读的格式，并返回一个由值和单位指示器组成的字符串。根据值的大小，单位部分可以是 `bytes`、`KiB`（kibibytes）、`MiB`（mebibytes）、`GiB`（gibibytes）、`TiB`（tebibytes）或 `PiB`（pebibytes）。

##### 参数

+   `bytes TEXT`：要格式化的字节计数。

##### 返回值

一个 `TEXT` 值。

##### 示例

```sql
mysql> SELECT sys.format_bytes(512), sys.format_bytes(18446644073709551615);
+-----------------------+----------------------------------------+
| sys.format_bytes(512) | sys.format_bytes(18446644073709551615) |
+-----------------------+----------------------------------------+
| 512 bytes             | 16383.91 PiB                           |
+-----------------------+----------------------------------------+
```
