# 15.1.11 ALTER VIEW Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-view.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-view.html)

```sql
ALTER
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    [DEFINER = *user*]
    [SQL SECURITY { DEFINER | INVOKER }]
    VIEW *view_name* [(*column_list*)]
    AS *select_statement*
    [WITH [CASCADED | LOCAL] CHECK OPTION]
```

这个语句改变了一个必须存在的视图的定义。语法与`CREATE VIEW`（请参见第 15.1.23 节，“CREATE VIEW Statement”）类似。这个语句需要视图的`CREATE VIEW`和`DROP`权限，并且对`SELECT`语句中引用的每一列都需要一些权限。只有定义者或具有`SET_USER_ID`权限（或已弃用的`SUPER`权限）的用户才被允许执行`ALTER VIEW`。
