# 5.6 常见查询示例

> 原文：[`dev.mysql.com/doc/refman/8.0/en/examples.html`](https://dev.mysql.com/doc/refman/8.0/en/examples.html)

5.6.1 列的最大值

5.6.2 某列最大值所在的行

5.6.3 每组列的最大值

5.6.4 某列的分组最大值所在的行

5.6.5 使用用户定义变量

5.6.6 使用外键

5.6.7 在两个键上搜索

5.6.8 计算每天的访问量

5.6.9 使用 AUTO_INCREMENT

这里是如何使用 MySQL 解决一些常见问题的示例。

一些示例使用表`shop`来保存每个文章（物品编号）对于某些交易商（经销商）的价格。假设每个交易商对于每篇文章有一个固定价格，那么(`article`, `dealer`)是记录的主键。

启动命令行工具**mysql**并选择一个数据库：

```sql
$> mysql *your-database-name*
```

要创建和填充示例表，请使用以下语句：

```sql
CREATE TABLE shop (
    article INT UNSIGNED  DEFAULT '0000' NOT NULL,
    dealer  CHAR(20)      DEFAULT ''     NOT NULL,
    price   DECIMAL(16,2) DEFAULT '0.00' NOT NULL,
    PRIMARY KEY(article, dealer));
INSERT INTO shop VALUES
    (1,'A',3.45),(1,'B',3.99),(2,'A',10.99),(3,'B',1.45),
    (3,'C',1.69),(3,'D',1.25),(4,'D',19.95);
```

执行完这些语句后，表应该包含以下内容：

```sql
SELECT * FROM shop ORDER BY article;

+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|       1 | A      |  3.45 |
|       1 | B      |  3.99 |
|       2 | A      | 10.99 |
|       3 | B      |  1.45 |
|       3 | C      |  1.69 |
|       3 | D      |  1.25 |
|       4 | D      | 19.95 |
+---------+--------+-------+
```
