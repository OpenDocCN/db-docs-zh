> 原文：[`dev.mysql.com/doc/refman/8.0/en/subquery-errors.html`](https://dev.mysql.com/doc/refman/8.0/en/subquery-errors.html)

#### 15.2.15.10 子查询错误

有一些错误仅适用于子查询。本节描述了这些错误。

+   不支持的子查询语法：

    ```sql
    ERROR 1235 (ER_NOT_SUPPORTED_YET)
    SQLSTATE = 42000
    Message = "This version of MySQL doesn't yet support
    'LIMIT & IN/ALL/ANY/SOME subquery'"
    ```

    这意味着 MySQL 不支持以下类似语句：

    ```sql
    SELECT * FROM t1 WHERE s1 IN (SELECT s2 FROM t2 ORDER BY s1 LIMIT 1)
    ```

+   子查询返回的列数不正确：

    ```sql
    ERROR 1241 (ER_OPERAND_COL)
    SQLSTATE = 21000
    Message = "Operand should contain 1 column(s)"
    ```

    在这种情况下会发生此错误：

    ```sql
    SELECT (SELECT column1, column2 FROM t2) FROM t1;
    ```

    如果子查询返回多列用于行比较，则可以使用子查询。在其他情境下，子查询必须是标量操作数。参见第 15.2.15.5 节，“行子查询”。

+   子查询返回的行数不正确：

    ```sql
    ERROR 1242 (ER_SUBSELECT_NO_1_ROW)
    SQLSTATE = 21000
    Message = "Subquery returns more than 1 row"
    ```

    对于子查询必须返回最多一行但返回多行的语句会发生此错误。考虑以下示例：

    ```sql
    SELECT * FROM t1 WHERE column1 = (SELECT column1 FROM t2);
    ```

    如果 `SELECT column1 FROM t2` 只返回一行，则前面的查询有效。如果子查询返回多于一行，则会出现错误 1242。在这种情况下，查询应重写为：

    ```sql
    SELECT * FROM t1 WHERE column1 = ANY (SELECT column1 FROM t2);
    ```

+   子查询中错误使用表：

    ```sql
    Error 1093 (ER_UPDATE_TABLE_USED)
    SQLSTATE = HY000
    Message = "You can't specify target table 'x'
    for update in FROM clause"
    ```

    在尝试在子查询中修改表并从同一表中进行选择的情况下会发生此错误：

    ```sql
    UPDATE t1 SET column2 = (SELECT MAX(column1) FROM t1);
    ```

    您可以使用公共表达式或派生表来解决此问题。参见第 15.2.15.12 节，“子查询限制”。

在 MySQL 8.0.19 及更高版本中，当在子查询中使用`TABLE`时，本节描述的所有错误也适用。

对于事务性存储引擎，子查询失败会导致整个语句失败。对于非事务性存储引擎，在遇到错误之前进行的数据修改会被保留。
