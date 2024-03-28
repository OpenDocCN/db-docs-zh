# 15.1.4 ALTER FUNCTION Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-function.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-function.html)

```sql
ALTER FUNCTION *func_name* [*characteristic* ...]

*characteristic*: {
    COMMENT '*string*'
  | LANGUAGE SQL
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
}
```

此语句可用于更改存储函数的特性。`ALTER FUNCTION`语句中可以指定多个更改。但是，您不能使用此语句更改存储函数的参数或主体；要进行此类更改，必须使用`DROP FUNCTION`和`CREATE FUNCTION`删除并重新创建函数。

对于该函数，您必须拥有`ALTER ROUTINE`权限。（该权限会自动授予函数创建者。）如果启用了二进制日志记录，`ALTER FUNCTION`语句可能还需要`SUPER`权限，如第 27.7 节“存储程序二进制日志记录”中所述。
