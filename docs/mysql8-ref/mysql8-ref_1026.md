> 原文：[`dev.mysql.com/doc/refman/8.0/en/declare-cursor.html`](https://dev.mysql.com/doc/refman/8.0/en/declare-cursor.html)

#### 15.6.6.2 游标 DECLARE 语句

```sql
DECLARE *cursor_name* CURSOR FOR *select_statement*
```

此语句声明一个游标，并将其与检索游标要遍历的行的`SELECT`语句相关联。要稍后获取行，请使用`FETCH`语句。`SELECT`语句检索的列数必须与`FETCH`语句中指定的输出变量数相匹配。

`SELECT`语句不能有`INTO`子句。

游标声明必须出现在处理程序声明之前，并且在变量和条件声明之后。

一个存储程序可能包含多个游标声明，但在给定块中声明的每个游标必须具有唯一名称。例如，请参阅第 15.6.6 节，“游标”。

通过`SHOW`语句提供的信息，在许多情况下可以通过使用带有`INFORMATION_SCHEMA`表的游标获得等效信息。
