> 原文：[`dev.mysql.com/doc/refman/8.0/en/leave.html`](https://dev.mysql.com/doc/refman/8.0/en/leave.html)

#### 15.6.5.4 `LEAVE` 语句

```sql
LEAVE *label*
```

此语句用于退出具有给定标签的流程控制结构。如果标签是最外层的存储程序块，`LEAVE` 将退出程序。

`LEAVE` 可以在 `BEGIN ... END` 或循环结构（`LOOP`、`REPEAT`、`WHILE`）中使用。

例如，请参阅 Section 15.6.5.5, “LOOP Statement”。
