> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-format-time.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-format-time.html)

#### 30.4.5.6 `format_time()` 函数

注意

从 MySQL 8.0.16 开始，`format_time()` Function") 已被弃用，并将在未来的 MySQL 版本中移除。使用它的应用程序应该迁移到使用内置的 `FORMAT_PICO_TIME()` 函数。参见 Section 14.21, “Performance Schema Functions”

给定一个以皮秒为单位的性能模式延迟或等待时间，将其转换为人类可读的格式，并返回一个由值和单位指示器组成的字符串。根据值的大小，单位部分可以是 `ps`（皮秒）、`ns`（纳秒）、`us`（微秒）、`ms`（毫秒）、`s`（秒）、`m`（分钟）、`h`（小时）、`d`（天）或 `w`（周）。

##### 参数

+   `picoseconds TEXT`：要格式化的皮秒值。

##### 返回值

一个 `TEXT` 值。

##### 示例

```sql
mysql> SELECT sys.format_time(3501), sys.format_time(188732396662000);
+-----------------------+----------------------------------+
| sys.format_time(3501) | sys.format_time(188732396662000) |
+-----------------------+----------------------------------+
| 3.50 ns               | 3.15 m                           |
+-----------------------+----------------------------------+
```
