> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-limit.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-limit.html)

#### 19.5.1.18 复制和 LIMIT

在`DELETE`、`UPDATE`和`INSERT ... SELECT`语句中，基于语句的复制`LIMIT`子句是不安全的，因为受影响的行的顺序未定义。（只有当这些语句还包含`ORDER BY`子句时，才能通过基于语句的复制正确复制这些语句。）当遇到这样的语句时：

+   在使用`STATEMENT`模式时，现在会发出警告，说明该语句对于基于语句的复制不安全。

    在使用`STATEMENT`模式时，即使 DML 语句包含`LIMIT`并且还有`ORDER BY`子句（因此变得确定性），也会发出警告。这是一个已知问题。（Bug #42851）

+   在使用`MIXED`模式时，该语句现在会自动使用基于行的模式进行复制。
