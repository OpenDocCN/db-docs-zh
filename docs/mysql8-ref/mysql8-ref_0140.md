# 5.6.2 持有某一列最大值的行

> 原文：[`dev.mysql.com/doc/refman/8.0/en/example-maximum-row.html`](https://dev.mysql.com/doc/refman/8.0/en/example-maximum-row.html)

*任务：找到最贵文章的编号、经销商和价格。*

这可以通过子查询轻松完成：

```sql
SELECT article, dealer, price
FROM   shop
WHERE  price=(SELECT MAX(price) FROM shop);

+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|    0004 | D      | 19.95 |
+---------+--------+-------+
```

另一个解决方案是使用`LEFT JOIN`，如下所示：

```sql
SELECT s1.article, s1.dealer, s1.price
FROM shop s1
LEFT JOIN shop s2 ON s1.price < s2.price
WHERE s2.article IS NULL;
```

你也可以通过按价格降序排序所有行，并使用 MySQL 特定的`LIMIT`子句仅获取第一行，像这样：

```sql
SELECT article, dealer, price
FROM shop
ORDER BY price DESC
LIMIT 1;
```

注意

如果有几篇价格都是 19.95 的最贵文章，使用`LIMIT`解决方案只会显示其中一篇。
