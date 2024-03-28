# 15.6.8 条件处理的限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/condition-handling-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/condition-handling-restrictions.html)

`SIGNAL`，`RESIGNAL`和`GET DIAGNOSTICS`不能作为预处理语句。例如，以下语句是无效的：

```sql
PREPARE stmt1 FROM 'SIGNAL SQLSTATE "02000"';
```

在类`'04'`中的`SQLSTATE`值不会被特殊处理。它们与其他异常一样处理。

在标准 SQL 中，第一个条件与前一个 SQL 语句返回的`SQLSTATE`值相关。在 MySQL 中，这并不保证，所以要获取主要错误，你不能这样做：

```sql
GET DIAGNOSTICS CONDITION 1 @errno = MYSQL_ERRNO;
```

相反，应该这样做：

```sql
GET DIAGNOSTICS @cno = NUMBER;
GET DIAGNOSTICS CONDITION @cno @errno = MYSQL_ERRNO;
```
