# 15.5.2 执行语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/execute.html`](https://dev.mysql.com/doc/refman/8.0/en/execute.html)

```sql
EXECUTE *stmt_name*
    [USING @*var_name* [, @*var_name*] ...]
```

在使用 `PREPARE` 准备语句之后，您可以使用一个 `EXECUTE` 语句来执行它，该语句引用了准备好的语句名称。如果准备的语句包含任何参数标记，您必须提供一个 `USING` 子句，列出包含要绑定到参数的值的用户变量。参数值只能由用户变量提供，并且 `USING` 子句必须命名与语句中参数标记数量完全相同的变量。

您可以多次执行给定的准备语句，向其传递不同的变量或在每次执行之前设置变量为不同的值。

有关示例，请参见 Section 15.5, “Prepared Statements”。
