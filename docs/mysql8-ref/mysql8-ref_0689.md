# 12.8.1 在 SQL 语句中使用 COLLATE

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-collate.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-collate.html)

使用`COLLATE`子句，您可以覆盖比较的默认排序规则。`COLLATE`可以在 SQL 语句的各个部分中使用。以下是一些示例：

+   在`ORDER BY`中：

    ```sql
    SELECT k
    FROM t1
    ORDER BY k COLLATE latin1_german2_ci;
    ```

+   在`AS`中：

    ```sql
    SELECT k COLLATE latin1_german2_ci AS k1
    FROM t1
    ORDER BY k1;
    ```

+   在`GROUP BY`中：

    ```sql
    SELECT k
    FROM t1
    GROUP BY k COLLATE latin1_german2_ci;
    ```

+   在聚合函数中：

    ```sql
    SELECT MAX(k COLLATE latin1_german2_ci)
    FROM t1;
    ```

+   在`DISTINCT`中：

    ```sql
    SELECT DISTINCT k COLLATE latin1_german2_ci
    FROM t1;
    ```

+   在`WHERE`中：

    ```sql
    SELECT *
    FROM t1
    WHERE _latin1 'Müller' COLLATE latin1_german2_ci = k;
    ```

    ```sql
    SELECT *
    FROM t1
    WHERE k LIKE _latin1 'Müller' COLLATE latin1_german2_ci;
    ```

+   在`HAVING`中：

    ```sql
    SELECT k
    FROM t1
    GROUP BY k
    HAVING k = _latin1 'Müller' COLLATE latin1_german2_ci;
    ```
