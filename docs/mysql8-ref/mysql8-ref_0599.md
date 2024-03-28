# 10.8.4 获取命名连接的执行计划信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/explain-for-connection.html`](https://dev.mysql.com/doc/refman/8.0/en/explain-for-connection.html)

要获取在命名连接中执行的可解释语句的执行计划，请使用此语句：

```sql
EXPLAIN [*options*] FOR CONNECTION *connection_id*;
```

`EXPLAIN FOR CONNECTION`返回当前用于在给定连接中执行查询的`EXPLAIN`信息。由于数据（及支持统计数据）的更改，它可能产生与在等效查询文本上运行`EXPLAIN`不同的结果。这种行为上的差异在诊断更瞬时的性能问题时可能很有用。例如，如果您在一个会话中运行一个需要很长时间才能完成的语句，使用另一个会话中的`EXPLAIN FOR CONNECTION`可能会提供有关延迟原因的有用信息。

*`connection_id`* 是连接标识符，可从`INFORMATION_SCHEMA` `PROCESSLIST`表或`SHOW PROCESSLIST`语句中获取。如果您拥有`PROCESS`权限，可以指定任何连接的标识符。否则，只能指定自己连接的标识符。在所有情况下，您必须具有足够的权限来解释指定连接上的查询。

如果命名连接未执行语句，则结果为空。否则，只有在命名连接中执行的语句是可解释的情况下，`EXPLAIN FOR CONNECTION`才适用。这包括`SELECT`, `DELETE`, `INSERT`, `REPLACE`, 和 `UPDATE`。（但是，`EXPLAIN FOR CONNECTION`不适用于准备语句，即使是这些类型的准备语句。）

如果命名连接正在执行可解释语句，则输出与在语句本身上使用`EXPLAIN`获得的相同。

如果命名连接正在执行不可解释的语句，则会发生错误。例如，您不能命名当前会话的连接标识符，因为`EXPLAIN`不可解释：

```sql
mysql> SELECT CONNECTION_ID();
+-----------------+
| CONNECTION_ID() |
+-----------------+
|             373 |
+-----------------+
1 row in set (0.00 sec)

mysql> EXPLAIN FOR CONNECTION 373;
ERROR 1889 (HY000): EXPLAIN FOR CONNECTION command is supported
only for SELECT/UPDATE/INSERT/DELETE/REPLACE
```

`Com_explain_other`状态变量指示执行的`EXPLAIN FOR CONNECTION`语句的数量。
