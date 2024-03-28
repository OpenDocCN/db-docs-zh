> 原文：[`dev.mysql.com/doc/refman/8.0/en/xa-states.html`](https://dev.mysql.com/doc/refman/8.0/en/xa-states.html)

#### 15.3.8.2 XA 事务状态

一个 XA 事务会经历以下状态：

1.  使用 `XA START` 来启动一个 XA 事务并将其置于 `ACTIVE` 状态。

1.  对于一个 `ACTIVE` 的 XA 事务，发出构成事务的 SQL 语句，然后发出 `XA END` 语句。`XA END` 将事务置于 `IDLE` 状态。

1.  对于一个 `IDLE` 的 XA 事务，可以发出 `XA PREPARE` 语句或 `XA COMMIT ... ONE PHASE` 语句：

    +   `XA PREPARE` 将事务置于 `PREPARED` 状态。此时，`XA RECOVER` 语句会在输出中包含事务的 *`xid`* 值，因为 `XA RECOVER` 列出所有处于 `PREPARED` 状态的 XA 事务。

    +   `XA COMMIT ... ONE PHASE` 准备并提交事务。*`xid`* 值不会被 `XA RECOVER` 列出，因为事务已经终止。

1.  对于一个 `PREPARED` 的 XA 事务，可以发出 `XA COMMIT` 语句来提交并终止事务，或者发出 `XA ROLLBACK` 来回滚并终止事务。

下面是一个简单的 XA 事务，作为全局事务的一部分向表中插入一行：

```sql
mysql> XA START 'xatest';
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO mytable (i) VALUES(10);
Query OK, 1 row affected (0.04 sec)

mysql> XA END 'xatest';
Query OK, 0 rows affected (0.00 sec)

mysql> XA PREPARE 'xatest';
Query OK, 0 rows affected (0.00 sec)

mysql> XA COMMIT 'xatest';
Query OK, 0 rows affected (0.00 sec)
```

在 MySQL 8.0.28 及更早版本中，在给定客户端连接的上下文中，XA 事务和本地（非 XA）事务是互斥的。例如，如果已经发出 `XA START` 来开始一个 XA 事务，那么在 XA 事务提交或回滚之前不能启动本地事务。反之，如果已经使用 `START TRANSACTION` 开始了本地事务，那么在事务提交或回滚之前不能使用 XA 语句。

MySQL 8.0.29 及更高版本支持分离的 XA 事务，通过`xa_detach_on_prepare`系统变量启用（默认为`ON`）。分离的事务在执行`XA PREPARE`后与当前会话断开连接（可以由另一个连接提交或回滚）。这意味着当前会话可以自由开始新的本地事务或 XA 事务，而无需等待准备的 XA 事务被提交或回滚。

当 XA 事务被分离时，一个连接对其准备的任何 XA 事务没有特殊知识。如果当前会话尝试提交或回滚给定的 XA 事务（即使是它准备的），在另一个连接已经这样做之后，请求将被拒绝，并显示无效的 XID 错误（`ER_XAER_NOTA`），因为请求的*`xid`*不再存在。

注意

分离的 XA 事务不能使用临时表。

当禁用分离的 XA 事务（`xa_detach_on_prepare`设置为`OFF`）时，一个 XA 事务会保持连接，直到由发起连接提交或回滚，就像之前描述的 MySQL 8.0.28 及更早版本一样。不建议为用于组复制的 MySQL 服务器实例禁用分离的 XA 事务；有关更多信息，请参阅服务器实例配置。

如果一个 XA 事务处于`ACTIVE`状态，你不能发出任何导致隐式提交的语句。这将违反 XA 协议，因为你无法回滚 XA 事务。尝试执行这样的语句会引发以下错误：

```sql
ERROR 1399 (XAE07): XAER_RMFAIL: The command cannot be executed
when global transaction is in the ACTIVE state
```

适用于前述备注的语句列在第 15.3.3 节，“导致隐式提交的语句”中。
