# 14.20.4 命名窗口

> 原文：[`dev.mysql.com/doc/refman/8.0/en/window-functions-named-windows.html`](https://dev.mysql.com/doc/refman/8.0/en/window-functions-named-windows.html)

可以通过在`OVER`子句中引用定义和命名窗口来定义窗口。为此，请使用`WINDOW`子句。如果在查询中存在，`WINDOW`子句位于`HAVING`和`ORDER BY`子句的位置之间，并具有以下语法：

```sql
WINDOW *window_name* AS (*window_spec*)
    [, *window_name* AS (*window_spec*)] ...
```

对于每个窗口定义，*`window_name`*是窗口名称，*`window_spec`*与`OVER`子句括号中给定的窗口规范类型相同，如第 14.20.2 节，“窗口函数概念和语法”中所述：

```sql
*window_spec*:
    [*window_name*] [*partition_clause*] [*order_clause*] [*frame_clause*]
```

对于多个`OVER`子句本应定义相同窗口的查询，`WINDOW`子句非常有用。相反，您可以一次定义窗口，为其命名，并在`OVER`子句中引用该名称。考虑以下查询，该查询多次定义相同窗口：

```sql
SELECT
  val,
  ROW_NUMBER() OVER (ORDER BY val) AS 'row_number',
  RANK()       OVER (ORDER BY val) AS 'rank',
  DENSE_RANK() OVER (ORDER BY val) AS 'dense_rank'
FROM numbers;
```

通过使用`WINDOW`一次性定义窗口并在`OVER`子句中引用窗口名称，可以更简单地编写查询：

```sql
SELECT
  val,
  ROW_NUMBER() OVER w AS 'row_number',
  RANK()       OVER w AS 'rank',
  DENSE_RANK() OVER w AS 'dense_rank'
FROM numbers
WINDOW w AS (ORDER BY val);
```

命名窗口还使得更容易尝试窗口定义以查看对查询结果的影响。您只需修改`WINDOW`子句中的窗口定义，而不是多个`OVER`子句定义。

如果`OVER`子句使用`OVER (*`window_name`* ...)`而不是`OVER *`window_name`*`，则可以通过添加其他子句修改命名窗口。例如，此查询定义了一个包含分区的窗口，并在`OVER`子句中使用`ORDER BY`以不同方式修改窗口：

```sql
SELECT
  DISTINCT year, country,
  FIRST_VALUE(year) OVER (w ORDER BY year ASC) AS first,
  FIRST_VALUE(year) OVER (w ORDER BY year DESC) AS last
FROM sales
WINDOW w AS (PARTITION BY country);
```

`OVER`子句只能向命名窗口添加属性，而不能修改它们。如果命名窗口定义包括分区、排序或帧属性，则引用窗口名称的`OVER`子句也不能包括相同类型的属性，否则将出现错误：

+   这种构造是允许的，因为窗口定义和引用的`OVER`子句不包含相同类型的属性：

    ```sql
    OVER (w ORDER BY country)
    ... WINDOW w AS (PARTITION BY country)
    ```

+   这种构造是不允许的，因为`OVER`子句为已经具有`PARTITION BY`的命名窗口指定了`PARTITION BY`：

    ```sql
    OVER (w PARTITION BY year)
    ... WINDOW w AS (PARTITION BY country)
    ```

命名窗口的定义本身可以以*`window_name`*开头。在这种情况下，允许前向和后向引用，但不允许循环：

+   这是允许的；它包含前向和后向引用，但没有循环：

    ```sql
    WINDOW w1 AS (w2), w2 AS (), w3 AS (w1)
    ```

+   这是不允许的，因为它包含一个循环：

    ```sql
    WINDOW w1 AS (w2), w2 AS (w3), w3 AS (w1)
    ```
