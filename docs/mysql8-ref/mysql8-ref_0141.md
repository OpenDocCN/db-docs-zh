# -   5.6.3 每组的列最大值

> -   原文：[`dev.mysql.com/doc/refman/8.0/en/example-maximum-column-group.html`](https://dev.mysql.com/doc/refman/8.0/en/example-maximum-column-group.html)

-   *任务：找到每篇文章的最高价格。*

```sql
SELECT article, MAX(price) AS price
FROM   shop
GROUP BY article
ORDER BY article;

+---------+-------+
| article | price |
+---------+-------+
|    0001 |  3.99 |
|    0002 | 10.99 |
|    0003 |  1.69 |
|    0004 | 19.95 |
+---------+-------+
```
