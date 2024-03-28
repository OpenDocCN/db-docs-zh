> 原文：[`dev.mysql.com/doc/refman/8.0/en/xa-statements.html`](https://dev.mysql.com/doc/refman/8.0/en/xa-statements.html)

#### 15.3.8.1 XA 事务 SQL 语句

要在 MySQL 中执行 XA 事务，请使用以下语句：

```sql
XA {START|BEGIN} *xid* [JOIN|RESUME]

XA END *xid* [SUSPEND [FOR MIGRATE]]

XA PREPARE *xid*

XA COMMIT *xid* [ONE PHASE]

XA ROLLBACK *xid*

XA RECOVER [CONVERT XID]
```

对于 `XA START`，`JOIN` 和 `RESUME` 子句被识别但没有效果。

对于 `XA END`，`SUSPEND [FOR MIGRATE]` 子句被识别但没有效果。

每个 XA 语句都以 `XA` 关键字开头，大多数需要一个 *`xid`* 值。*`xid`* 是一个 XA 事务标识符。它指示语句适用于哪个事务。*`xid`* 值由客户端提供，或由 MySQL 服务器生成。*`xid`* 值有一到三部分：

```sql
*xid*: *gtrid* [, *bqual* [, *formatID* ]]
```

*`gtrid`* 是全局事务标识符，*`bqual`* 是分支限定符，*`formatID`* 是标识 *`gtrid`* 和 *`bqual`* 值使用的格式的数字。如语法所示，*`bqual`* 和 *`formatID`* 是可选的。如果未给出，默认 *`bqual`* 值为 `''`。如果未给出，默认 *`formatID`* 值为 1。

*`gtrid`* 和 *`bqual`* 必须是字符串字面量，每个最多 64 字节（而非字符）长。*`gtrid`* 和 *`bqual`* 可以以几种方式指定。您可以使用带引号的字符串（`'ab'`）、十六进制字符串（`X'6162'`、`0x6162`）或位值（`b'*`nnnn`*'`）。

*`formatID`* 是一个无符号整数。

*`gtrid`* 和 *`bqual`* 值由 MySQL 服务器底层的 XA 支持程序解释为字节。然而，在解析包含 XA 语句的 SQL 语句时，服务器使用特定的字符集。为了安全起见，将 *`gtrid`* 和 *`bqual`* 写成十六进制字符串。

*`xid`* 值通常由事务管理器生成。一个事务管理器生成的值必须与其他事务管理器生成的值不同。给定的事务管理器必须能够在 `XA RECOVER` 语句返回的值列表中识别出自己的 *`xid`* 值。

`XA START *`xid`*` 使用给定的 *`xid`* 值启动一个 XA 事务。每个 XA 事务必须具有唯一的 *`xid`* 值，因此该值不能当前被另一个 XA 事务使用。使用 *`gtrid`* 和 *`bqual`* 值来评估唯一性。随后的所有 XA 语句必须使用与 `XA START` 语句中给定的 *`xid`* 值相同的 *`xid`* 值指定。如果您使用这些语句之一，但指定的 *`xid`* 值与某个现有 XA 事务不对应，则会发生错误。

从 MySQL 8.0.31 开始，当服务器运行时带有 `--replicate-do-db` 或 `--replicate-ignore-db` 时，`XA START`、`XA BEGIN`、`XA END`、`XA COMMIT` 和 `XA ROLLBACK` 语句不会被默认数据库过滤。

一个或多个 XA 事务可以是同一个全局事务的一部分。给定全局事务中的所有 XA 事务必须在 *`xid`* 值中使用相同的 *`gtrid`* 值。因此，*`gtrid`* 值必须是全局唯一的，以确保对于给定 XA 事务是哪个全局事务的一部分没有歧义。 *`xid`* 值的 *`bqual`* 部分必须对于全局事务中的每个 XA 事务都不同。 （*`bqual`* 值必须不同的要求是当前 MySQL XA 实现的限制。它不是 XA 规范的一部分。）

`XA RECOVER` 语句返回 MySQL 服务器上处于`PREPARED`状态的 XA 事务的信息。（参见 Section 15.3.8.2, “XA 事务状态”.）输出包括服务器上每个这样的 XA 事务的行，无论是哪个客户端启动的。

`XA RECOVER` 需要 `XA_RECOVER_ADMIN` 权限。这个权限要求阻止用户发现除了自己之外的未完成准备好的 XA 事务的 XID 值。它不影响 XA 事务的正常提交或回滚，因为启动它的用户知道它的 XID。

`XA RECOVER` 输出行如下（对于由部分 `'abc'`、`'def'` 和 `7` 组成的 *`xid`* 值的示例）：

```sql
mysql> XA RECOVER;
+----------+--------------+--------------+--------+
| formatID | gtrid_length | bqual_length | data   |
+----------+--------------+--------------+--------+
|        7 |            3 |            3 | abcdef |
+----------+--------------+--------------+--------+
```

输出列的含义如下：

+   `formatID` 是事务 *`xid`* 的 *`formatID`* 部分

+   `gtrid_length` 是 *`xid`* 的 *`gtrid`* 部分的字节长度

+   `bqual_length` 是 *`xid`* 的 *`bqual`* 部分的字节长度

+   `data` 是 *`xid`* 的 *`gtrid`* 和 *`bqual`* 部分的串联

XID 值可能包含不可打印字符。`XA RECOVER` 允许一个可选的 `CONVERT XID` 子句，以便客户端可以请求十六进制的 XID 值。
