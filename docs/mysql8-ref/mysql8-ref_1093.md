> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-errors.html`](https://dev.mysql.com/doc/refman/8.0/en/show-errors.html)

#### 15.7.7.17 SHOW ERRORS Statement

```sql
SHOW ERRORS [LIMIT [*offset*,] *row_count*]
SHOW COUNT(*) ERRORS
```

`SHOW ERRORS`是一条诊断语句，类似于`SHOW WARNINGS`，不同之处在于它仅显示错误的信息，而不是错误、警告和注释。

`LIMIT`子句的语法与`SELECT`语句相同。请参见 Section 15.2.13, “SELECT Statement”。

`SHOW COUNT(*) ERRORS`语句显示错误的数量。您还可以从`error_count`变量中检索此数字：

```sql
SHOW COUNT(*) ERRORS;
SELECT @@error_count;
```

`SHOW ERRORS`和`error_count`仅适用于错误，而不是警告或注释。在其他方面，它们类似于`SHOW WARNINGS`和`warning_count`。特别是，`SHOW ERRORS`无法显示超过`max_error_count`条消息的信息，如果错误数量超过`max_error_count`，则`error_count`的值可以超过`max_error_count`的值。

有关更多信息，请参见 Section 15.7.7.42, “SHOW WARNINGS Statement”。
