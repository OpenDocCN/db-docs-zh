> 原文：[`dev.mysql.com/doc/refman/8.0/en/close.html`](https://dev.mysql.com/doc/refman/8.0/en/close.html)

#### 15.6.6.1 游标关闭语句

```sql
CLOSE *cursor_name*
```

此语句关闭先前打开的游标。示例请参见第 15.6.6 节，“游标”。

如果游标未打开，则会发生错误。

如果未显式关闭，游标将在声明它的`BEGIN ... END`块结束时关闭。
