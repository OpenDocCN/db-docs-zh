> 原文：[`dev.mysql.com/doc/refman/8.0/en/fetch.html`](https://dev.mysql.com/doc/refman/8.0/en/fetch.html)

#### 15.6.6.3 Cursor FETCH Statement

```sql
FETCH [[NEXT] FROM] *cursor_name* INTO *var_name* [, *var_name*] ...
```

该语句获取与指定游标关联的 `SELECT` 语句的下一行，并移动游标指针。如果存在一行，则获取的列将存储在命名变量中。`SELECT` 语句检索的列数必须与 `FETCH` 语句中指定的输出变量数相匹配。

如果没有更多的行可用，则会发生一个 SQLSTATE 值为 `'02000'` 的 No Data 条件。要检测这种情况，您可以为其设置一个处理程序（或者为 `NOT FOUND` 条件）。例如，请参见 Section 15.6.6, “Cursors”。

请注意，另一个操作，例如 `SELECT` 或另一个 `FETCH`，也可能通过引发相同的条件来导致处理程序执行。如果需要区分哪个操作引发了条件，请将操作放置在其自己的 `BEGIN ... END` 块中，以便将其与自己的处理程序关联起来。
