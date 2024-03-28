> 原文：[`dev.mysql.com/doc/refman/8.0/en/lateral-derived-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/lateral-derived-tables.html)

#### 15.2.15.9 侧向派生表

通常，派生表不能引用同一`FROM`子句中之前表的列。从 MySQL 8.0.14 开始，可以将派生表定义为侧向派生表，以指定允许这种引用。

非侧向派生表使用第 15.2.15.8 节“派生表”中讨论的语法来指定。侧向派生表的语法与非侧向派生表相同，只是在派生表规范之前指定了关键字`LATERAL`。`LATERAL`关键字必须在每个要用作侧向派生表的表之前。

侧向派生表受到以下限制：

+   侧向派生表只能出现在`FROM`子句中，可以是用逗号分隔的表列表，也可以是连接规范（`JOIN`、`INNER JOIN`、`CROSS JOIN`、`LEFT [OUTER] JOIN`或`RIGHT [OUTER] JOIN`）中。

+   如果一个侧向派生表在连接子句的右操作数中，并且包含对左操作数的引用，则连接操作必须是`INNER JOIN`、`CROSS JOIN`或`LEFT [OUTER] JOIN`。

    如果表在左操作数中，并且包含对右操作数的引用，则连接操作必须是`INNER JOIN`、`CROSS JOIN`或`RIGHT [OUTER] JOIN`。

+   如果一个侧向派生表引用了一个聚合函数，则该函数的聚合查询不能是包含侧向派生表的`FROM`子句所属的查询。

+   根据 SQL 标准，MySQL 始终将与表函数（如`JSON_TABLE()`）的连接视为已使用`LATERAL`。这在 MySQL 的任何版本中都是正确的，这就是为什么即使在 MySQL 8.0.14 之前的版本中也可以针对此函数进行连接。在 MySQL 8.0.14 及更高版本中，`LATERAL`关键字是隐式的，并且不允许在`JSON_TABLE()`之前使用。这也符合 SQL 标准。

以下讨论显示了侧向派生表如何使得某些 SQL 操作成为可能，这些操作无法通过非侧向派生表完成，或者需要更低效的解决方法。

假设我们想解决这个问题：给定一个销售团队成员的表（其中每行描述一个销售团队成员），以及所有销售的表（其中每行描述一笔销售：销售人员、客户、金额、日期），确定每个销售人员的最大销售额及其客户。这个问题可以有两种方法来解决。

解决问题的第一种方法：对于每个销售人员，计算最大销售额，并找到提供此最大销售额的客户。在 MySQL 中，可以这样做：

```sql
SELECT
  salesperson.name,
  -- find maximum sale size for this salesperson
  (SELECT MAX(amount) AS amount
    FROM all_sales
    WHERE all_sales.salesperson_id = salesperson.id)
  AS amount,
  -- find customer for this maximum size
  (SELECT customer_name
    FROM all_sales
    WHERE all_sales.salesperson_id = salesperson.id
    AND all_sales.amount =
         -- find maximum size, again
         (SELECT MAX(amount) AS amount
           FROM all_sales
           WHERE all_sales.salesperson_id = salesperson.id))
  AS customer_name
FROM
  salesperson;
```

那个查询是低效的，因为它在每个销售人员中计算最大尺寸两次（在第一个子查询中一次，在第二个子查询中一次）。

我们可以尝试通过在每个销售人员中计算最大值并在派生表中“缓存”它来实现效率提升，如这个修改后的查询所示：

```sql
SELECT
  salesperson.name,
  max_sale.amount,
  max_sale_customer.customer_name
FROM
  salesperson,
  -- calculate maximum size, cache it in transient derived table max_sale
  (SELECT MAX(amount) AS amount
    FROM all_sales
    WHERE all_sales.salesperson_id = salesperson.id)
  AS max_sale,
  -- find customer, reusing cached maximum size
  (SELECT customer_name
    FROM all_sales
    WHERE all_sales.salesperson_id = salesperson.id
    AND all_sales.amount =
        -- the cached maximum size
        max_sale.amount)
  AS max_sale_customer;
```

然而，在 SQL-92 中，该查询是非法的，因为派生表不能依赖于同一 `FROM` 子句中的其他表。派生表必须在查询的持续时间内保持恒定，不能包含对其他 `FROM` 子句表列的引用。如此编写的查询会产生以下错误：

```sql
ERROR 1054 (42S22): Unknown column 'salesperson.id' in 'where clause'
```

在 SQL:1999 中，如果派生表前面有 `LATERAL` 关键字（表示“这个派生表依赖于其左侧的先前表”），则查询变得合法：

```sql
SELECT
  salesperson.name,
  max_sale.amount,
  max_sale_customer.customer_name
FROM
  salesperson,
  -- calculate maximum size, cache it in transient derived table max_sale
  LATERAL
  (SELECT MAX(amount) AS amount
    FROM all_sales
    WHERE all_sales.salesperson_id = salesperson.id)
  AS max_sale,
  -- find customer, reusing cached maximum size
  LATERAL
  (SELECT customer_name
    FROM all_sales
    WHERE all_sales.salesperson_id = salesperson.id
    AND all_sales.amount =
        -- the cached maximum size
        max_sale.amount)
  AS max_sale_customer;
```

一个侧向派生表不需要是恒定的，并且每当依赖的前一个表中的新行被顶层查询处理时，它就会被更新。

解决问题的第二种方法：如果 `SELECT` 列表中的子查询可以返回多列，则可以使用不同的解决方案：

```sql
SELECT
  salesperson.name,
  -- find maximum size and customer at same time
  (SELECT amount, customer_name
    FROM all_sales
    WHERE all_sales.salesperson_id = salesperson.id
    ORDER BY amount DESC LIMIT 1)
FROM
  salesperson;
```

那是高效的但是非法的。它不起作用，因为这样的子查询只能返回单列：

```sql
ERROR 1241 (21000): Operand should contain 1 column(s)
```

重写查询的一种尝试是从派生表中选择多列：

```sql
SELECT
  salesperson.name,
  max_sale.amount,
  max_sale.customer_name
FROM
  salesperson,
  -- find maximum size and customer at same time
  (SELECT amount, customer_name
    FROM all_sales
    WHERE all_sales.salesperson_id = salesperson.id
    ORDER BY amount DESC LIMIT 1)
  AS max_sale;
```

然而，那也不起作用。派生表依赖于 `salesperson` 表，因此在没有 `LATERAL` 的情况下失败：

```sql
ERROR 1054 (42S22): Unknown column 'salesperson.id' in 'where clause'
```

添加 `LATERAL` 关键字使查询合法：

```sql
SELECT
  salesperson.name,
  max_sale.amount,
  max_sale.customer_name
FROM
  salesperson,
  -- find maximum size and customer at same time
  LATERAL
  (SELECT amount, customer_name
    FROM all_sales
    WHERE all_sales.salesperson_id = salesperson.id
    ORDER BY amount DESC LIMIT 1)
  AS max_sale;
```

简而言之，`LATERAL` 是刚刚讨论的两种方法中所有缺点的高效解决方案。
