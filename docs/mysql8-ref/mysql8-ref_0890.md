# 15.1.7 ALTER PROCEDURE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-procedure.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-procedure.html)

```sql
ALTER PROCEDURE *proc_name* [*characteristic* ...]

*characteristic*: {
    COMMENT '*string*'
  | LANGUAGE SQL
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
}
```

这个语句可以用来改变存储过程的特性。一个`ALTER PROCEDURE`语句中可以指定多个更改。然而，你不能使用这个语句来改变存储过程的参数或主体；要进行这样的更改，你必须使用`DROP PROCEDURE`和`CREATE PROCEDURE`来删除并重新创建该存储过程。

你必须拥有`ALTER ROUTINE`权限才能操作该存储过程。默认情况下，该权限会自动授予给存储过程的创建者。可以通过禁用`automatic_sp_privileges`系统变量来改变这种行为。参见 Section 27.2.2, “Stored Routines and MySQL Privileges”。
