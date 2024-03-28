# 5.6.4 持有某一列分组最大值的行

> 原文：[`dev.mysql.com/doc/refman/8.0/en/example-maximum-column-group-row.html`](https://dev.mysql.com/doc/refman/8.0/en/example-maximum-column-group-row.html)

*任务：对于每篇文章，找到价格最高的经销商或经销商。*

这个问题可以通过这样的子查询来解决：

```sql
SELECT article, dealer, price
FROM   shop s1
WHERE  price=(SELECT MAX(s2.price)
              FROM shop s2
              WHERE s1.article = s2.article)
ORDER BY article;

+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|    0001 | B      |  3.99 |
|    0002 | A      | 10.99 |
|    0003 | C      |  1.69 |
|    0004 | D      | 19.95 |
+---------+--------+-------+
```

前面的示例使用了相关子查询，这可能效率低下（参见 Section 15.2.15.7, “Correlated Subqueries”）。解决问题的其他可能性包括在 `FROM` 子句中使用无关联子查询、`LEFT JOIN` 或具有窗口函数的公共表达式。

无关联子查询：

```sql
SELECT s1.article, dealer, s1.price
FROM shop s1
JOIN (
  SELECT article, MAX(price) AS price
  FROM shop
  GROUP BY article) AS s2
  ON s1.article = s2.article AND s1.price = s2.price
ORDER BY article;
```

`LEFT JOIN`：

```sql
SELECT s1.article, s1.dealer, s1.price
FROM shop s1
LEFT JOIN shop s2 ON s1.article = s2.article AND s1.price < s2.price
WHERE s2.article IS NULL
ORDER BY s1.article;
```

`LEFT JOIN` 的工作原理是，当 `s1.price` 达到最大值时，没有比它更大的 `s2.price`，因此对应的 `s2.article` 值为 `NULL`。参见 Section 15.2.13.2, “JOIN Clause”。

具有窗口函数的公共表达式：

```sql
WITH s1 AS (
   SELECT article, dealer, price,
          RANK() OVER (PARTITION BY article
                           ORDER BY price DESC
                      ) AS `Rank`
     FROM shop
)
SELECT article, dealer, price
  FROM s1
  WHERE `Rank` = 1
ORDER BY article;
```
