# 15.1.29 DROP PROCEDURE and DROP FUNCTION Statements

> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-procedure.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-procedure.html)

```sql
DROP {PROCEDURE | FUNCTION} [IF EXISTS] *sp_name*
```

这些语句用于删除存储例程（存储过程或函数）。也就是说，指定的例程将从服务器中移除。(`DROP FUNCTION` 也用于删除可加载函数；参见 Section 15.7.4.2, “DROP FUNCTION Statement for Loadable Functions”.)

要删除存储例程，您必须具有 `ALTER ROUTINE` 权限。（如果启用了 `automatic_sp_privileges` 系统变量，则在创建例程时自动授予该权限和 `EXECUTE` 给例程创建者，并在删除例程时从创建者那里撤销。请参见 Section 27.2.2, “Stored Routines and MySQL Privileges”.)

另外，如果例程的定义者具有 `SYSTEM_USER` 权限，则删除它的用户也必须具有此权限。这在 MySQL 8.0.16 及更高版本中执行。

`IF EXISTS` 子句是 MySQL 的扩展。如果存储过程或函数不存在，它可以防止错误发生。会产生一个警告，可以通过 `SHOW WARNINGS` 查看。

`DROP FUNCTION` 也用于删除可加载函数（参见 Section 15.7.4.2, “DROP FUNCTION Statement for Loadable Functions”).
