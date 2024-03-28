> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-subqueries.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-subqueries.html)

#### 15.2.15.11 优化子查询

开发仍在进行中，因此长期来看，没有可靠的优化提示。以下列表提供了一些有趣的技巧，您可能想尝试一下。另请参阅 Section 10.2.2，“优化子查询、派生表、视图引用和公共表达式”。

+   将子查询外部的子句移到内部。例如，使用这个查询：

    ```sql
    SELECT * FROM t1
      WHERE s1 IN (SELECT s1 FROM t1 UNION ALL SELECT s1 FROM t2);
    ```

    而不是这个查询：

    ```sql
    SELECT * FROM t1
      WHERE s1 IN (SELECT s1 FROM t1) OR s1 IN (SELECT s1 FROM t2);
    ```

    举个例子，使用这个查询：

    ```sql
    SELECT (SELECT column1 + 5 FROM t1) FROM t2;
    ```

    而不是这个查询：

    ```sql
    SELECT (SELECT column1 FROM t1) + 5 FROM t2;
    ```
