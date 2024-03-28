> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-format-statement.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-format-statement.html)

#### 30.4.5.5 The format_statement() Function

给定一个字符串（通常表示一个 SQL 语句），将其缩短到`statement_truncate_len`配置选项指定的长度，并返回结果。如果字符串短于`statement_truncate_len`，则不会进行截断。否则，字符串的中间部分将被省略号（`...`）替换。

此函数对从性能模式表中检索的可能较长的语句进行格式化，使其达到已知的固定最大长度。

##### 参数

+   `statement LONGTEXT`：需要格式化的语句。

##### 配置选项

`format_statement()` Function") 操作可以使用以下配置选项或其对应的用户定义变量进行修改（参见第 30.4.2.1 节，“sys_config 表”）：

+   `statement_truncate_len`，`@sys.statement_truncate_len`

    `format_statement()` Function") 函数返回的语句的最大长度。超过此长度的语句将被截断至此长度。默认值为 64。

##### 返回值

一个`LONGTEXT`值。

##### 示例

默认情况下，`format_statement()` Function") 将语句截断为不超过 64 个字符。设置`@sys.statement_truncate_len`会改变当前会话的截断长度：

```sql
mysql> SET @stmt = 'SELECT variable, value, set_time, set_by FROM sys_config';
mysql> SELECT sys.format_statement(@stmt);
+----------------------------------------------------------+
| sys.format_statement(@stmt)                              |
+----------------------------------------------------------+
| SELECT variable, value, set_time, set_by FROM sys_config |
+----------------------------------------------------------+
mysql> SET @sys.statement_truncate_len = 32;
mysql> SELECT sys.format_statement(@stmt);
+-----------------------------------+
| sys.format_statement(@stmt)       |
+-----------------------------------+
| SELECT variabl ... ROM sys_config |
+-----------------------------------+
```
