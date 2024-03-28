# 14.4.3 逻辑运算符

> 原文：[`dev.mysql.com/doc/refman/8.0/en/logical-operators.html`](https://dev.mysql.com/doc/refman/8.0/en/logical-operators.html)

**表 14.5 逻辑运算符**

| 名称 | 描述 |
| --- | --- |
| `AND`, `&&` | 逻辑 AND |
| `NOT`, `!` | 取反值 |
| `OR`, `&#124;&#124;` | 逻辑 OR |
| `XOR` | 逻辑 XOR |

在 SQL 中，所有逻辑运算符的计算结果为`TRUE`、`FALSE`或`NULL`（`UNKNOWN`）。在 MySQL 中，这些分别实现为 1（`TRUE`）、0（`FALSE`）和`NULL`。大部分内容适用于不同的 SQL 数据库服务器，尽管有些服务器可能会返回任何非零值作为`TRUE`。

MySQL 将任何非零、非`NULL`值计算为`TRUE`。例如，以下语句都评估为`TRUE`：

```sql
mysql> SELECT 10 IS TRUE;
-> 1
mysql> SELECT -10 IS TRUE;
-> 1
mysql> SELECT 'string' IS NOT NULL;
-> 1
```

+   `NOT`, `!`

    逻辑 NOT。如果操作数为`0`，则计算结果为`1`，如果操作数为非零，则为`0`，`NOT NULL` 返回`NULL`。

    ```sql
    mysql> SELECT NOT 10;
     -> 0
    mysql> SELECT NOT 0;
     -> 1
    mysql> SELECT NOT NULL;
     -> NULL
    mysql> SELECT ! (1+1);
     -> 0
    mysql> SELECT ! 1+1;
     -> 1
    ```

    最后一个示例产生`1`，因为该表达式的计算方式与`(!1)+1`相同。

    `!` 运算符是 MySQL 的非标准扩展。从 MySQL 8.0.17 开始，此运算符已被弃用；预计在未来的 MySQL 版本中将不再支持。应用程序应调整为使用标准 SQL `NOT` 运算符。

+   `AND`, `&&`

    逻辑 AND。如果所有操作数均为非零且非`NULL`，则计算结果为`1`，如果一个或多个操作数为`0`，则结果为`0`，否则返回`NULL`。

    ```sql
    mysql> SELECT 1 AND 1;
     -> 1
    mysql> SELECT 1 AND 0;
     -> 0
    mysql> SELECT 1 AND NULL;
     -> NULL
    mysql> SELECT 0 AND NULL;
     -> 0
    mysql> SELECT NULL AND 0;
     -> 0
    ```

    `&&` 运算符是 MySQL 的非标准扩展。从 MySQL 8.0.17 开始，此运算符已被弃用；预计在未来的 MySQL 版本中将不再支持。应用程序应调整为使用标准 SQL `AND` 运算符。

+   `OR`, `||`

    逻辑 OR。当两个操作数均为非`NULL`时，如果任一操作数为非零，则结果为`1`，否则为`0`。如果有一个操作数为`NULL`，则结果为`1`，如果另一个操作数为非零，则为`NULL`。如果两个操作数均为`NULL`，则结果为`NULL`。

    ```sql
    mysql> SELECT 1 OR 1;
     -> 1
    mysql> SELECT 1 OR 0;
     -> 1
    mysql> SELECT 0 OR 0;
     -> 0
    mysql> SELECT 0 OR NULL;
     -> NULL
    mysql> SELECT 1 OR NULL;
     -> 1
    ```

    注意

    如果启用了 `PIPES_AS_CONCAT` SQL 模式，则 `||` 表示 SQL 标准的字符串连接运算符（类似于 `CONCAT()`）。

    `||` 运算符是 MySQL 的非标准扩展。从 MySQL 8.0.17 开始，该运算符已被弃用；预计在未来的 MySQL 版本中将移除对其的支持。应用程序应调整为使用标准 SQL 的 `OR` 运算符。例外情况：如果启用了 `PIPES_AS_CONCAT`，则不适用弃用规则，因为在这种情况下，`||` 表示字符串连接。

+   `XOR`

    逻辑异或。如果任一操作数为 `NULL`，则返回 `NULL`。对于非 `NULL` 操作数，如果奇数个操作数为非零，则评估为 `1`，否则返回 `0`。

    ```sql
    mysql> SELECT 1 XOR 1;
     -> 0
    mysql> SELECT 1 XOR 0;
     -> 1
    mysql> SELECT 1 XOR NULL;
     -> NULL
    mysql> SELECT 1 XOR 1 XOR 1;
     -> 1
    ```

    `a XOR b` 在数学上等同于 `(a AND (NOT b)) OR ((NOT a) and b)`。
