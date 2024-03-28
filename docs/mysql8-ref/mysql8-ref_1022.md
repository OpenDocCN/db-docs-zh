> 原文：[`dev.mysql.com/doc/refman/8.0/en/return.html`](https://dev.mysql.com/doc/refman/8.0/en/return.html)

#### 15.6.5.7 RETURN Statement

```sql
RETURN *expr*
```

`RETURN` 语句终止存储函数的执行，并将值 *`expr`* 返回给函数调用者。存储函数中必须至少有一个 `RETURN` 语句。如果函数有多个退出点，则可能有多个。

此语句不在存储过程、触发器或事件中使用。`LEAVE` 语句可用于退出这些类型的存储程序。
