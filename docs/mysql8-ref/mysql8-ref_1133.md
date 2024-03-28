# 15.8.4 USE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/use.html`](https://dev.mysql.com/doc/refman/8.0/en/use.html)

```sql
USE *db_name*
```

`USE`语句告诉 MySQL 使用命名的数据库作为后续语句的默认（当前）数据库。此语句需要数据库或其中某个对象的一些权限。

在会话结束或发出另一个`USE`语句之前，命名的数据库将保持默认状态：

```sql
USE db1;
SELECT COUNT(*) FROM mytable;   # selects from db1.mytable
USE db2;
SELECT COUNT(*) FROM mytable;   # selects from db2.mytable
```

数据库名称必须在单行上指定。数据库名称中不支持换行符。

通过`USE`语句将特定数据库设置为默认数据库并不妨碍访问其他数据库中的表。以下示例访问了`db1`数据库中的`author`表和`db2`数据库中的`editor`表：

```sql
USE db1;
SELECT author_name,editor_name FROM author,db2.editor
  WHERE author.editor_id = db2.editor.editor_id;
```
