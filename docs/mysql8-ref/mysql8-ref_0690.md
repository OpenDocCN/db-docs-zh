# 12.8.2 COLLATE 子句优先级

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-collate-precedence.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-collate-precedence.html)

`COLLATE`子句具有很高的优先级（高于`||`），因此以下两个表达式是等价的：

```sql
x || y COLLATE z
x || (y COLLATE z)
```
