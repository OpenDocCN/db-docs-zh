> 原文：[`dev.mysql.com/doc/refman/8.0/en/temporary-table-problems.html`](https://dev.mysql.com/doc/refman/8.0/en/temporary-table-problems.html)

#### B.3.6.2 TEMPORARY 表问题

使用`CREATE TEMPORARY TABLE`创建的临时表有以下限制：

+   `TEMPORARY`表仅受`InnoDB`，`MEMORY`，`MyISAM`和`MERGE`存储引擎支持。

+   NDB Cluster 不支持临时表。

+   `SHOW TABLES`语句不会列出`TEMPORARY`表。

+   要重命名`TEMPORARY`表，`RENAME TABLE`不起作用。请改用`ALTER TABLE`：

    ```sql
    ALTER TABLE old_name RENAME new_name;
    ```

+   你不能在同一查询中多次引用`TEMPORARY`表。例如，以下内容不起作用：

    ```sql
    SELECT * FROM temp_table JOIN temp_table AS t2;
    ```

    该语句会产生以下错误：

    ```sql
    ERROR 1137: Can't reopen table: 'temp_table'
    ```

    如果您的查询允许使用公共表达式（CTE）而不是`TEMPORARY`表，则可以解决此问题。例如，以下内容会因无法重新打开表的错误而失败：

    ```sql
    CREATE TEMPORARY TABLE t SELECT 1 AS col_a, 2 AS col_b;
    SELECT * FROM t AS t1 JOIN t AS t2;
    ```

    要避免错误，请使用定义 CTE 的`WITH`")子句，而不是`TEMPORARY`表：

    ```sql
    WITH cte AS (SELECT 1 AS col_a, 2 AS col_b)
    SELECT * FROM cte AS t1 JOIN cte AS t2;
    ```

+   如果在存储函数中多次引用临时表且使用不同别名，即使引用发生在函数内的不同语句中，也会出现无法重新打开表的错误。这可能发生在在存储函数外创建的临时表，并在多个调用和被调用函数中引用时。

+   如果使用与现有非`TEMPORARY`表相同名称创建`TEMPORARY`表，则在删除`TEMPORARY`表之前，非`TEMPORARY`表将被隐藏，即使表使用不同存储引擎。

+   在使用复制时使用临时表存在已知问题。有关更多信息，请参阅 Section 19.5.1.31, “Replication and Temporary Tables”。
