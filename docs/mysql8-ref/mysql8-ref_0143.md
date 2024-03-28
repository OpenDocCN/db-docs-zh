# 5.6.5 使用用户定义变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/example-user-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/example-user-variables.html)

你可以使用 MySQL 用户变量来记住结果，而无需将它们存储在客户端的临时变量中。（参见第 11.4 节，“用户定义变量”.）

例如，要找到价格最高和最低的文章，可以这样做：

```sql
mysql> SELECT @min_price:=MIN(price),@max_price:=MAX(price) FROM shop;
mysql> SELECT * FROM shop WHERE price=@min_price OR price=@max_price;
+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|    0003 | D      |  1.25 |
|    0004 | D      | 19.95 |
+---------+--------+-------+
```

注意

也可以将数据库对象的名称（如表或列）存储在用户变量中，然后在 SQL 语句中使用这个变量；但是，这需要使用准备语句。更多信息请参见第 15.5 节，“准备语句”。
