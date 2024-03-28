# 11.1.6 布尔字面值

> 原文：[`dev.mysql.com/doc/refman/8.0/en/boolean-literals.html`](https://dev.mysql.com/doc/refman/8.0/en/boolean-literals.html)

常量`TRUE`和`FALSE`分别求值为`1`和`0`。常量名称可以以任何大小写形式编写。

```sql
mysql> SELECT TRUE, true, FALSE, false;
 -> 1, 1, 0, 0
```
