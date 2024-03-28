> 原文：[`dev.mysql.com/doc/refman/8.0/en/exists-and-not-exists-subqueries.html`](https://dev.mysql.com/doc/refman/8.0/en/exists-and-not-exists-subqueries.html)

#### 15.2.15.6 带有 EXISTS 或 NOT EXISTS 的子查询

如果子查询返回任何行，`EXISTS *`子查询`*` 是 `TRUE`，而 `NOT EXISTS *`子查询`*` 是 `FALSE`。例如：

```sql
SELECT column1 FROM t1 WHERE EXISTS (SELECT * FROM t2);
```

传统上，`EXISTS` 子查询以 `SELECT *` 开头，但也可以以 `SELECT 5` 或 `SELECT column1` 或任何其他内容开头。MySQL 会忽略这种子查询中的 `SELECT` 列表，因此不会有任何区别。

对于前面的示例，如果 `t2` 包含任何行，即使行中只有 `NULL` 值，`EXISTS` 条件也为 `TRUE`。这实际上是一个不太可能的示例，因为 `[NOT] EXISTS` 子查询几乎总是包含相关性。以下是一些更现实的示例：

+   一个或多个城市中存在什么样的商店？

    ```sql
    SELECT DISTINCT store_type FROM stores
      WHERE EXISTS (SELECT * FROM cities_stores
                    WHERE cities_stores.store_type = stores.store_type);
    ```

+   没有城市中存在什么样的商店？

    ```sql
    SELECT DISTINCT store_type FROM stores
      WHERE NOT EXISTS (SELECT * FROM cities_stores
                        WHERE cities_stores.store_type = stores.store_type);
    ```

+   所有城市中存在什么样的商店？

    ```sql
    SELECT DISTINCT store_type FROM stores
      WHERE NOT EXISTS (
        SELECT * FROM cities WHERE NOT EXISTS (
          SELECT * FROM cities_stores
           WHERE cities_stores.city = cities.city
           AND cities_stores.store_type = stores.store_type));
    ```

最后一个示例是一个双重嵌套的 `NOT EXISTS` 查询。也就是说，它在一个 `NOT EXISTS` 子句中有一个 `NOT EXISTS` 子句。形式上，它回答了“是否存在一个城市有一个不在 `Stores` 中的商店”这个问题？但更容易说的是，嵌套的 `NOT EXISTS` 回答了“对于所有 *`y`*，*`x`* 是否都为 `TRUE`？”

在 MySQL 8.0.19 及更高版本中，您还可以在子查询中使用 `NOT EXISTS` 或 `NOT EXISTS` 与 `TABLE`，就像这样：

```sql
SELECT column1 FROM t1 WHERE EXISTS (TABLE t2);
```

结果与在子查询中没有 `WHERE` 子句的情况下使用 `SELECT *` 相同。
