# 15.3.2 无法回滚的语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/cannot-roll-back.html`](https://dev.mysql.com/doc/refman/8.0/en/cannot-roll-back.html)

有些语句是无法回滚的。一般来说，这些包括数据定义语言（DDL）语句，比如创建或删除数据库的语句，创建、删除或修改表或存储过程的语句。

你应该设计你的事务不包括这样的语句。如果你在事务早期发出一个无法回滚的语句，然后稍后另一个语句失败，那么在这种情况下，通过发出一个`ROLLBACK`语句无法完全回滚事务的效果。
