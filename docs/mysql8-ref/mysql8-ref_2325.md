> 原文：[`dev.mysql.com/doc/refman/8.0/en/using-date.html`](https://dev.mysql.com/doc/refman/8.0/en/using-date.html)

#### B.3.4.2 使用 DATE 列时的问题

`DATE` 值的格式为 `'*`YYYY-MM-DD`*'`。根据标准 SQL，不允许使用其他格式。你应该在 `UPDATE` 表达式和 `SELECT` 语句的 `WHERE` 子句中使用此格式。例如：

```sql
SELECT * FROM t1 WHERE date >= '2003-05-05';
```

作为便利，MySQL 会自动将日期转换为数字，如果日期在数字上下文中使用，反之亦然。MySQL 还允许在更新和将日期与 `DATE`、`DATETIME` 或 `TIMESTAMP` 列进行比较的 `WHERE` 子句中使用“宽松”的字符串格式。 “宽松”格式意味着任何标点字符都可以用作部分之间的分隔符。例如，`'2004-08-15'` 和 `'2004#08#15'` 是等效的。MySQL 还可以转换不包含分隔符的字符串（例如 `'20040815'`），只要它作为日期是有意义的。

当你使用 `<`, `<=`, `=`, `>=`, `>`, 或 `BETWEEN` 操作符将 `DATE`、`TIME`、`DATETIME` 或 `TIMESTAMP` 与常量字符串进行比较时，MySQL 通常会将字符串转换为内部长整型以加快比较速度（也为了更“宽松”的字符串检查）。然而，此转换受以下例外情况影响：

+   当你比较两列时

+   当你将 `DATE`、`TIME`、`DATETIME` 或 `TIMESTAMP` 列与表达式进行比较时

+   当你使用除了刚列出的方法之外的任何比较方法，如 `IN` 或 `STRCMP()`。

对于这些例外情况，通过将对象转换为字符串并执行字符串比较来进行比较。

为了安全起见，假设字符串是作为字符串进行比较的，并且如果你想将时间值与字符串进行比较，则使用适��的字符串函数。

特殊的“零”日期 `'0000-00-00'` 可以存储和检索为 `'0000-00-00'`。当通过 Connector/ODBC 使用 `'0000-00-00'` 日期时，它会自动转换为 `NULL`，因为 ODBC 无法处理这种日期。

因为 MySQL 执行了刚才描述的转换，所以下面的语句有效（假设 `idate` 是一个 `DATE` 列）:

```sql
INSERT INTO t1 (idate) VALUES (19970505);
INSERT INTO t1 (idate) VALUES ('19970505');
INSERT INTO t1 (idate) VALUES ('97-05-05');
INSERT INTO t1 (idate) VALUES ('1997.05.05');
INSERT INTO t1 (idate) VALUES ('1997 05 05');
INSERT INTO t1 (idate) VALUES ('0000-00-00');

SELECT idate FROM t1 WHERE idate >= '1997-05-05';
SELECT idate FROM t1 WHERE idate >= 19970505;
SELECT MOD(idate,100) FROM t1 WHERE idate >= 19970505;
SELECT idate FROM t1 WHERE idate >= '19970505';
```

然而，以下语句不起作用：

```sql
SELECT idate FROM t1 WHERE STRCMP(idate,'20030505')=0;
```

`STRCMP()` 是一个字符串函数，因此它将 `idate` 转换为 `'*`YYYY-MM-DD`*'` 格式的字符串并执行字符串比较。它不会将 `'20030505'` 转换为日期 `'2003-05-05'` 并执行日期比较。

如果你启用了`ALLOW_INVALID_DATES` SQL 模式，MySQL 允许你存储仅进行有限检查的日期：MySQL 仅要求日期在 1 到 31 的范围内，月份在 1 到 12 的范围内。这使得 MySQL 在 Web 应用程序中非常方便，其中你在三个不同字段中获取年、月和日，并且想要存储用户插入的内容（无需日期验证）。

MySQL 允许你存储日期，其中日期或月份和日期为零。如果你想要在 `DATE` 列中存储出生日期，并且只知道日期的一部分，这是很方便的。要禁止日期中的零月或日部分，请启用 `NO_ZERO_IN_DATE` 模式。

MySQL 允许你将“零”值 `'0000-00-00'` 存储为“虚拟日期”。在某些情况下，这比使用 `NULL` 值更方便。如果要存储在 `DATE` 列中的日期无法转换为任何合理值，MySQL 将存储 `'0000-00-00'`。要禁止 `'0000-00-00'`，请启用 `NO_ZERO_DATE` 模式。

要让 MySQL 检查所有日期并仅接受合法日期（除非被 `IGNORE` 覆盖），请将 `sql_mode` 系统变量设置为 `"NO_ZERO_IN_DATE,NO_ZERO_DATE"`。
