# 5.6.1 列的最大值

> 原文：[`dev.mysql.com/doc/refman/8.0/en/example-maximum-column.html`](https://dev.mysql.com/doc/refman/8.0/en/example-maximum-column.html)

“最高的项目编号是多少？”

```sql
SELECT MAX(article) AS article FROM shop;

+---------+
| article |
+---------+
|       4 |
+---------+
```
